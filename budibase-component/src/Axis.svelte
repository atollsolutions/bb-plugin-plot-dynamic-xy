<script>
	import { onMount, afterUpdate, onDestroy } from 'svelte';
	import { scaleLinear } from 'd3-scale';
	
	export let points;
	export let scale_ob;
	export let points1;

	let svg;
	let size = 0.8;
	let width=350;
	let height=350;
	const padding = { top: 20, right: 20, bottom: 5, left: 25 };
	
	let xScale, yScale, xTicks, yTicks;
	console.log(scale_ob)
	function calculateScales() {
		let range = Math.max(scale_ob['max_x'] - scale_ob['min_x'], scale_ob['max_y'] - scale_ob['min_y']);
    let svgSize = window.innerWidth * size;
		xScale = scaleLinear()
			.domain([scale_ob['min_x'], scale_ob['max_x']])
			.range([padding.left, width - padding.right]);

		yScale = scaleLinear()
			.domain([scale_ob['min_y'], scale_ob['max_y']])
			.range([height - padding.bottom, padding.top]);

		xTicks = [];
		yTicks = [];

		for (let i=scale_ob['min_x']; i<=scale_ob['max_x']; i += parseFloat(scale_ob['nodx'])) {
			xTicks.push(i);
		}

		for (let i=scale_ob['min_y']; i<=scale_ob['max_y']; i += parseFloat(scale_ob['nody'])) {
			yTicks.push(i);
		}
	}



	afterUpdate(() => {
		calculateScales();
	});
	onMount(() => {
		calculateScales();
		resize();
	});
	onDestroy(() => {
		xScale = null;
		yScale = null;
		xTicks = null;
		yTicks = null;
	});

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
	}

</script>

<svelte:window on:resize="{resize}" />

<svg bind:this={svg} style="width: {window.innerWidth * size}px; height: {window.innerWidth * size}px">
	{#if xScale && yScale && xTicks && yTicks}
	<g class='axis y-axis'>
		{#each yTicks as tick}
			<g class='tick tick-{tick}' transform='translate(0, {yScale(tick)})'>
				<line x1='{padding.left}' x2='{xScale(scale_ob.max_x)}'/>
				<text x='{padding.left - 8}' y='+4'>{tick}</text>
			</g>
		{/each}
	</g>

	<g class='axis x-axis'>
		{#each xTicks as tick}
			<g class='tick' transform='translate({xScale(tick)},0)'>
				<line y1='{yScale(0)}' y2='{yScale(scale_ob.max_y)}'/>
				<text y='{height - padding.bottom + 16}'>{tick}</text>
			</g>
		{/each}
	</g>

	{#each points as point}
  {#if !(isNaN(point.x) || isNaN(point.y))}
    <circle cx='{xScale(point.x)}' cy='{yScale(point.y)}' r='5'/>
    <title>This point is static</title>
  {/if}
{/each}
	{#each points1 as pointt}
	 {#if !(isNaN(pointt.x) || isNaN(pointt.y))}
		<circle class='v1' cx='{xScale(pointt.x)}' cy='{yScale(pointt.y)}' r='5'/>
		<title>This point is dynamic</title>
		{/if}
	{/each}
	{/if}
</svg>

<style>
	svg {
		width: 100%;
		height: 100%;
		float: left;
	}

	circle {
		fill: orange;
		fill-opacity: 0.6;
		stroke: rgba(0,0,0,0.5);
	}

	.tick line {
		stroke: rgb(1, 4, 14);
		stroke-dasharray: 2;
	}

	text {
		font-size: 14px;
		fill: rgb(153, 153, 153);
	}

	.x-axis text {
		text-anchor: middle;
	}

	.y-axis text {
		text-anchor: end;
	}
</style>