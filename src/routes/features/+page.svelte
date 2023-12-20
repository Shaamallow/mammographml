<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	let loaded = false;

	let divPCA: HTMLDivElement;
	let divCorrelationMap: HTMLDivElement;
	let divTSNE: HTMLDivElement;

	let features_div: HTMLDivElement;

	let data_original: any;
	let data_pca: any;
	let data_tsne: any;
	let data_correlation_map: any;

	onMount(() => {
		loaded = true; // this is for the fade transition

		// Now load the dataset only ONCE and store it in the variables
		// to be used in the draw() functions later on and avoid loading
		// the dataset every time the window is resized (testing purposes especially...)
		d3.csv('/dataset.csv').then((data) => {
			console.log('data_original');
			data_original = data;
		});
		d3.csv('/dataset_pca.csv').then((data) => {
			console.log('data_pca');
			data_pca = [];
			let nRows = data.length;
			for (let i = 0; i < nRows; i++) {
				let row = data[i];
				data_pca.push({
					PC1: parseFloat(row.PC1),
					PC2: parseFloat(row.PC2),
					id: row.id,
					diagnosis: parseInt(row.diagnosis)
				});
			}
			console.log(data_pca);

			drawPCA();
		});
		d3.csv('/dataset_tsne.csv').then((data) => {
			console.log('data_tsne');
			data_tsne = [];
			let nRows = data.length;
			for (let i = 0; i < nRows; i++) {
				let row = data[i];
				data_tsne.push({
					x: parseFloat(row.X),
					y: parseFloat(row.Y),
					id: row.id,
					diagnosis: parseInt(row.diagnosis)
				});
			}

			console.log(data_tsne);
			drawTSNE();
		});
		d3.csv('/dataset_correlation_map.csv').then((data) => {
			data_correlation_map = data;
			console.log('data_correlation_map');
		});

		window.addEventListener('resize', draw);
	});

	function draw(): void {
		console.log('draw');

		// empty the corresponding svg elements
		d3.select(features_div).html(null);
		d3.select(divPCA).html(null);
		d3.select(divCorrelationMap).html(null);
		d3.select(divTSNE).html(null);

		// call the draw functions to fill the svg elements
		drawFeatures();
		drawPCA();
		drawCorrelationMap();
		drawTSNE();
	}

	function drawFeatures(): void {}
	function drawPCA(): void {
		// PCA function

		// get the width of parent element
		// check if divPCA is defined

		let width = divPCA.clientWidth;
		let height = divPCA.clientHeight;

		let margin = 50;

		let min_value_PC1 = d3.min(data_pca, (d: any) => d.PC1 as number) as number;
		let max_value_PC1 = d3.max(data_pca, (d: any) => d.PC1 as number) as number;

		const xScale = d3
			.scaleLinear()
			.domain([min_value_PC1 * 1.1, max_value_PC1 * 1.1])
			.range([margin, width - margin]);

		let min_value_PC2 = d3.min(data_pca, (d: any) => d.PC2 as number) as number;
		let max_value_PC2 = d3.max(data_pca, (d: any) => d.PC2 as number) as number;

		const yScale = d3
			.scaleLinear()
			.domain([min_value_PC2 * 1.1, max_value_PC2 * 1.1])
			.range([height - margin, margin]);

		const xAxis = d3.axisBottom(xScale);
		const yAxis = d3.axisLeft(yScale);

		const svg = d3.select(divPCA).append('svg').attr('width', width).attr('height', height);

		// X axis
		svg
			.append('g')
			.attr('transform', `translate(0, ${height - margin})`)
			.call(xAxis)
			.append('text')
			.attr('x', width / 2)
			.attr('y', margin - 20)
			.attr('text-anchor', 'middle')
			.text('Principal Component 1')
			.attr('class', 'fill-current');

		// Y axis
		svg
			.append('g')
			.attr('transform', `translate(${margin}, 0)`)
			.call(yAxis)
			.append('text')
			.attr('x', -height / 2)
			.attr('y', -margin + 20)
			.attr('text-anchor', 'middle')
			.attr('transform', 'rotate(-90)')
			.attr('class', 'fill-current')
			.text('Principal Component 2');

		// Add dots
		svg
			.append('g')
			.selectAll('circle')
			.data(data_pca)
			.enter()
			.append('circle')
			.attr('cx', (d: any) => xScale(d.PC1))
			.attr('cy', (d: any) => yScale(d.PC2))
			.attr('r', 5)
			.attr('class', (d: any) => (d.diagnosis == 0 ? 'fill-primary' : 'fill-error'))
			.attr('fill-opacity', 0.3);

		// add legend text on the right using a small circle and a text

		svg
			.append('circle')
			.attr('cx', width - margin * 2)
			.attr('cy', margin)
			.attr('r', 5)
			.attr('class', 'fill-primary');

		svg
			.append('circle')
			.attr('cx', width - margin * 2)
			.attr('cy', margin * 2)
			.attr('r', 5)
			.attr('class', 'fill-error');

		svg
			.append('text')
			.attr('x', width - margin * 1.8)
			.attr('y', margin)
			.text('Benign')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');
		svg
			.append('text')
			.attr('x', width - margin * 1.8)
			.attr('y', margin * 2)
			.text('Malignant')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');

		// add title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', margin / 2)
			.text('PCA Analysis')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}
	function drawTSNE(): void {
		// TSNE function
		// get the width of parent element

		let width = divTSNE.clientWidth;
		let height = divTSNE.clientHeight;

		let margin = 50;

		let min_value_x = d3.min(data_tsne, (d: any) => d.x as number) as number;
		let max_value_x = d3.max(data_tsne, (d: any) => d.x as number) as number;

		const xScale = d3
			.scaleLinear()
			.domain([min_value_x * 1.1, max_value_x * 1.1])
			.range([margin, width - margin]);

		let min_value_y = d3.min(data_tsne, (d: any) => d.y as number) as number;
		let max_value_y = d3.max(data_tsne, (d: any) => d.y as number) as number;

		const yScale = d3
			.scaleLinear()
			.domain([min_value_y * 1.1, max_value_y * 1.1])
			.range([height - margin, margin]);

		const xAxis = d3.axisBottom(xScale);
		const yAxis = d3.axisLeft(yScale);

		const svg = d3.select(divTSNE).append('svg').attr('width', width).attr('height', height);

		// X axis
		svg
			.append('g')
			.attr('transform', `translate(0, ${height - margin})`)
			.call(xAxis)
			.append('text')
			.attr('x', width / 2)
			.attr('y', margin - 20)
			.attr('text-anchor', 'middle')
			.attr('class', 'fill-current')
			.text('t-SNE Component 1');

		// Y axis
		svg
			.append('g')
			.attr('transform', `translate(${margin}, 0)`)
			.call(yAxis)
			.append('text')
			.attr('x', -height / 2)
			.attr('y', -margin + 20)
			.attr('text-anchor', 'middle')
			.attr('transform', 'rotate(-90)')
			.attr('class', 'fill-current')
			.text('t-SNE Component 2');

		// Add dots
		svg
			.append('g')
			.selectAll('circle')
			.data(data_tsne)
			.enter()
			.append('circle')
			.attr('cx', (d: any) => xScale(d.x))
			.attr('cy', (d: any) => yScale(d.y))
			.attr('r', 5)
			.attr('class', (d: any) => (d.diagnosis == 0 ? 'fill-primary' : 'fill-error'))
			.attr('fill-opacity', 0.3);

		// add legend text on the right using a small circle and a text

		svg
			.append('circle')
			.attr('cx', width - margin * 2)
			.attr('cy', margin)
			.attr('r', 5)
			.attr('class', 'fill-primary');

		svg
			.append('circle')
			.attr('cx', width - margin * 2)
			.attr('cy', margin * 2)
			.attr('r', 5)
			.attr('class', 'fill-error');

		svg
			.append('text')
			.attr('x', width - margin * 1.8)
			.attr('y', margin)
			.text('Benign')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');
		svg
			.append('text')
			.attr('x', width - margin * 1.8)
			.attr('y', margin * 2)
			.text('Malignant')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');

		// add title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', margin / 2)
			.text('TSNE Analysis')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}
	function drawCorrelationMap(): void {}
</script>

<svelte:head>
	<title>Dataset</title>
</svelte:head>

{#if loaded}
	<div in:fade={{ duration: 400, delay: 100 }} class="mx-auto">
		<div class="bg-base-100 py-10 px-4 mx-auto mb-10">
			<div
				class="flex flex-col items-center justify-stretch group max-w-3xl mx-auto overflow-visible"
			>
				<div class="md:flex block justify-center items-center md:mt-3 mb-6">
					<h1
						class="text-center font-display text-5xl md:text-7xl font-bold bg-clip-text bg-secondary anim-gradient pb-3"
					>
						Features Analysis
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center">Breast Cancer Wisconsin Dataset</div>
				</div>
				<div class="text-center max-w-md">
					Detailed analysis of the features of the dataset, and their respective distribution.
				</div>
			</div>
		</div>

		<!-- PCA | Correlation Map | TSNE -->

		<div
			class="md:w-[50rem] rounded mx-auto bg-neutral flex flex-col lg:flex-row lg:flew-wrap justify-center items-center gap-5 p-2 mt-10"
		>
			<div class="w-full h-[30rem] bg-primary" bind:this={divCorrelationMap}></div>
		</div>

		<div class="divider my-20" />

		<div
			class="container mx-auto flex flex-col lg:flex-row lg:flew-wrap justify-center items-center gap-5 p-2"
		>
			<div class="w-full h-[30rem] bg-neutral rounded-md p-2" bind:this={divPCA}></div>
			<div class="w-full h-[30rem] bg-neutral rounded-md p-2" bind:this={divTSNE}></div>
		</div>

		<div class="divider my-20" />

		<div
			class="mx-10 flex flex-col justify-center items-center gap-10"
			bind:this={features_div}
			id="features_div"
		>
			<div class="flex justify-center items-center gap-5">
				<h1 class="text-center font-display text-3xl">Feature 1</h1>
				<svg class="w-[50rem] bg-secondary h-[20rem]"></svg>
			</div>
			<div class="flex justify-center items-center gap-5">
				<h1 class="text-center font-display text-3xl">Feature 2</h1>
				<svg class="w-[50rem] bg-secondary h-[20rem]"></svg>
			</div>
		</div>

		<div class="divider m-10" />
	</div>
{/if}
