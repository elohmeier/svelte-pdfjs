<!-- @component
Render a page from a PDF document. Must be a child of a `Document` component.
 -->
<svelte:options immutable />

<!--
	@todo Immutable could be a bad idea since it would not update 
	for getViewport functions that are defined inline
	when their dependencies change.
 -->
<script context="module" lang="ts">
	import type { CalcViewport, MultipleOf90 } from '$lib/utils/target_dimension.js';
	import type { PDFDocumentProxy, PDFPageProxy } from 'pdfjs-dist';
	import type { PageViewport } from 'pdfjs-dist/types/src/display/display_utils.js';
	import { getContext, onDestroy, createEventDispatcher } from 'svelte';
	import type { Writable } from 'svelte/store';
	import PageCanvas from './PageInternals/PageCanvas.svelte';
</script>

<script lang="ts">
	const dispatch = createEventDispatcher();

	interface $$Events {
		/**
		 * Dispatched when the page is rendered.
		 */
		rendered: void;
	}

	// #region props
	/**
	 * The page number to show.
	 */
	export let num: number;
	/**
	 * The scale to show the PDF at.
	 * @default {1}
	 */
	export let scale: number = 1;
	/**
	 * Rotate the page by a multiple of 90 degrees.
	 * @default {0}
	 */
	export let rotation: MultipleOf90 = 0;
	/**
	 * Render a separate text layer (only for the canvas renderer.)
	 * @default {false}
	 */
	export let renderTextLayer: boolean = false;

	/**
	 * A callback invoked with the current page used to determine the viewport.
	 * Use this if you need something more complicated than the default based on scale.
	 */
	export let getViewport: CalcViewport | undefined = undefined;
	// #endregion props

	onDestroy(() => page?.cleanup());

	const current_doc: Writable<PDFDocumentProxy> = getContext('svelte_pdfjs_doc');

	let page: PDFPageProxy;
	let viewport: PageViewport;
	let canvas: PageCanvas;

	/* <========================================================================================> */

	$: $current_doc?.getPage(num).then((p) => (page = p));

	let _get_viewport: CalcViewport;
	$: _get_viewport = getViewport ?? ((p, r) => p.getViewport({ scale, rotation: r }));

	$: if (page) viewport = _get_viewport(page, rotation);

	export async function redraw() {
		await canvas.render_page();
	}

	export function get_canvas() {
		return canvas.get_canvas();
	}
</script>

<PageCanvas
	bind:this={canvas}
	{page}
	{viewport}
	render_text_layer={renderTextLayer}
	on:rendered={() => dispatch('rendered')}
/>
/>
