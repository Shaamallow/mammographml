<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	let loaded = false;

	let divPCA: HTMLDivElement;
	let divCorrelationMap: HTMLDivElement;
	let divTSNE: HTMLDivElement;

	let divFeatures: HTMLDivElement;

	let dataOriginal: any;
	let dataPCA: any;
	let dataTSNE: any;
	let dataCorrelationMap: any;

	onMount(() => {
		loaded = true; // this is for the fade transition

		const parseValuesToFloat = (obj: Record<string, string | number>): Record<string, number> => {
			const result: Record<string, number> = {};

			for (const key in obj) {
				if (obj.hasOwnProperty(key)) {
					// Parse the value to float if it's a string
					result[key] =
						typeof obj[key] === 'string' ? parseFloat(obj[key] as string) : (obj[key] as number);
				}
			}

			return result;
		};
		// Now load the dataset only ONCE and store it in the variables
		// to be used in the draw() functions later on and avoid loading
		// the dataset every time the window is resized (testing purposes especially...)
		d3.csv('/dataset.csv').then((data) => {
			console.log('data_original');
			// Parse the values to float
			dataOriginal = data.map(parseValuesToFloat);
		});
		d3.csv('/dataset_pca.csv').then((data) => {
			console.log('data_pca loaded');
			dataPCA = [];
			let nRows = data.length;
			for (let i = 0; i < nRows; i++) {
				let row = data[i];
				dataPCA.push({
					PC1: parseFloat(row.PC1),
					PC2: parseFloat(row.PC2),
					id: row.id,
					diagnosis: parseInt(row.diagnosis)
				});
			}

			drawPCA();
		});
		d3.csv('/dataset_tsne.csv').then((data) => {
			console.log('data_tsne loaded');
			dataTSNE = [];
			let nRows = data.length;
			for (let i = 0; i < nRows; i++) {
				let row = data[i];
				dataTSNE.push({
					x: parseFloat(row.X),
					y: parseFloat(row.Y),
					id: row.id,
					diagnosis: parseInt(row.diagnosis)
				});
			}

			drawTSNE();
		});
		d3.csv('/dataset_correlation_map.csv').then((data) => {
			console.log('data_correlation_map');

			dataCorrelationMap = data.map(parseValuesToFloat);

			// remove 1st column of each row
			dataCorrelationMap.forEach((row: any) => {
				delete row[''];
			});
			drawCorrelationMap();
		});

		window.addEventListener('resize', draw);
	});

	function draw(): void {
		console.log('draw');

		// empty the corresponding svg elements
		//d3.select(divFeatures).html(null);
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

		let min_value_PC1 = d3.min(dataPCA, (d: any) => d.PC1 as number) as number;
		let max_value_PC1 = d3.max(dataPCA, (d: any) => d.PC1 as number) as number;

		const xScale = d3
			.scaleLinear()
			.domain([min_value_PC1 * 1.1, max_value_PC1 * 1.1])
			.range([margin, width - margin]);

		let min_value_PC2 = d3.min(dataPCA, (d: any) => d.PC2 as number) as number;
		let max_value_PC2 = d3.max(dataPCA, (d: any) => d.PC2 as number) as number;

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
			.data(dataPCA)
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

		let min_value_x = d3.min(dataTSNE, (d: any) => d.x as number) as number;
		let max_value_x = d3.max(dataTSNE, (d: any) => d.x as number) as number;

		const xScale = d3
			.scaleLinear()
			.domain([min_value_x * 1.1, max_value_x * 1.1])
			.range([margin, width - margin]);

		let min_value_y = d3.min(dataTSNE, (d: any) => d.y as number) as number;
		let max_value_y = d3.max(dataTSNE, (d: any) => d.y as number) as number;

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
			.data(dataTSNE)
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
	function drawCorrelationMap(): void {
		// Correlation Map function

		// get the width of parent element
		let width = divCorrelationMap.clientWidth;
		let height = width; // to make it almost a square

		let margin = 10;
		let margin_right = 50;

		let nRows = dataCorrelationMap.length;
		let keys = Object.keys(dataCorrelationMap[0]);
		let nCols = keys.length;

		let min_value = d3.min(dataCorrelationMap, (d: any) => {
			let min = d3.min(Object.values(d), (d: any) => d as number);
			return min;
		}) as number;

		let max_value = d3.max(dataCorrelationMap, (d: any) => {
			let max = d3.max(Object.values(d), (d: any) => d as number);
			return max;
		}) as number;

		let colorScale = d3.scaleSequential(d3.interpolateViridis).domain([min_value, max_value]);

		let xScale = d3
			.scaleBand()
			.range([margin + width * 0.2, width - margin_right])
			.domain(d3.range(nCols) as any);

		let yScale = d3
			.scaleBand()
			.range([margin + height * 0.1, height * 0.85 - margin])
			.domain(d3.range(nRows) as any);

		const svg = d3
			.select(divCorrelationMap)
			.append('svg')
			.attr('width', width)
			.attr('height', height);

		let xAxis = d3.axisBottom(xScale).tickFormat((d: any) => keys[d]);
		let yAxis = d3.axisLeft(yScale).tickFormat((d: any) => keys[d]);

		// Now add the squares for the correlation map
		// each row of the dataset is a row of squares
		// each square is a rect element
		svg
			.selectAll()
			.data(
				dataCorrelationMap.flatMap((row: any, i: any) =>
					Object.entries(row).map(([key, value], j) => ({ i, j, value }))
				)
			)
			.enter()
			.append('rect')
			.attr('x', (d: any) => xScale(d.j) as number)
			.attr('y', (d: any) => yScale(d.i) as number)
			.attr('width', xScale.bandwidth())
			.attr('height', yScale.bandwidth())
			.style('fill', (d: any) => colorScale(d.value));

		// Add X axis
		svg
			.append('g')
			.attr('transform', `translate(0, ${height * 0.85 - margin / 2})`)
			.call(xAxis)
			.selectAll('text')
			.attr('transform', 'translate(-10,0)rotate(-45)')
			.attr('class', 'fill-current')
			.style('text-anchor', 'end');

		// Add Y axis
		svg
			.append('g')
			.attr('transform', `translate(${margin / 2 + width * 0.2}, 0)`)
			.call(yAxis)
			.selectAll('text')
			.attr('class', 'fill-current')
			.style('text-anchor', 'end');

		// add title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', height * 0.07)
			.text('Correlation Map')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}
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

		<div class="container lg:w-[50rem] mx-auto flex justify-center items-center mt-10 p-2">
			<div class="w-full m-2 bg-neutral rounded-md p-2" bind:this={divCorrelationMap}></div>
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
			bind:this={divFeatures}
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
