<script lang="ts">
	import { RenderingCancelledException } from 'pdfjs-dist';
	import type { PDFPageProxy, RenderTask } from 'pdfjs-dist';
	import type { PageViewport } from 'pdfjs-dist/types/src/display/display_utils.js';
	import { tick, createEventDispatcher } from 'svelte';
	import TextLayer from './TextLayer.svelte';

	export let page: PDFPageProxy;
	export let viewport: PageViewport;
	export let render_text_layer: boolean;

	let canvas: HTMLCanvasElement;

	let render_task: RenderTask;

	const dispatch = createEventDispatcher();

	interface $$Events {
		/**
		 * Dispatched when the page is rendered.
		 */
		rendered: CustomEvent<PDFPageProxy>;
	}

	export async function render_page() {
		render_task?.cancel();
		await tick();
		render_task = page.render({
			canvasContext: canvas.getContext('2d')!,
			viewport,
		});

		try {
			await render_task.promise;
			dispatch('rendered', page);
		} catch (err) {
			if (!(err instanceof RenderingCancelledException)) throw err;
		}
	}

	$: if (viewport && canvas) render_page();

	export function get_canvas() {
		return canvas;
	}
</script>

<div>
	<canvas
		bind:this={canvas}
		width={viewport?.width}
		height={viewport?.height}
	/>{#if render_text_layer}
		<TextLayer {page} {viewport} />
	{/if}
</div>

<style>
	div {
		position: relative;
		padding: 0;
	}

	canvas {
		display: block;
		margin: 0;
	}
</style>
