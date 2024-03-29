<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	let loaded = false;

	let divPCA: HTMLDivElement;
	let divCorrelationMap: HTMLDivElement;
	let divTSNE: HTMLDivElement;

	let divFeaturesMost: HTMLDivElement;
	let divFeaturesLeast: HTMLDivElement;

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
		d3.csv('/dataset/dataset.csv').then((data) => {
			// Parse the values to float
			dataOriginal = data.map(parseValuesToFloat);

			// remove 1st column
			dataOriginal.forEach((row: any) => {
				delete row[''];
			});

			drawFeatures();
		});
		d3.csv('/dataset/dataset_pca.csv').then((data) => {
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
		d3.csv('/dataset/dataset_tsne.csv').then((data) => {
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
		d3.csv('/dataset/dataset_correlation_map.csv').then((data) => {
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
		// empty the corresponding svg elements
		//d3.select(divFeatures).html(null);
		d3.select(divPCA).html(null);
		d3.select(divCorrelationMap).html(null);
		d3.select(divTSNE).html(null);
		d3.select(divFeaturesMost).html(null);
		d3.select(divFeaturesLeast).html(null);

		// call the draw functions to fill the svg elements
		drawFeatures();
		drawPCA();
		drawCorrelationMap();
		drawTSNE();
	}

	function kernelDensityEstimator(kernel: any, X: any) {
		return function (V: any) {
			return X.map(function (x: any) {
				return [
					x,
					d3.mean(V, function (v: any) {
						return kernel(x - v);
					})
				];
			});
		};
	}

	function kernelEpanechnikov(k: any) {
		return function (v: any) {
			return Math.abs((v /= k)) <= 1 ? (0.75 * (1 - v * v)) / k : 0;
		};
	}

	function drawFeatures(): void {
		// Create a density Plot for each feature
		// get the width of parent element

		let width = 0;
		if (window.innerWidth < 640) {
			width = divFeaturesMost.clientWidth;
		} else {
			width = divFeaturesMost.clientWidth * 0.8;
		}

		let height = 200;
		let margin = {
			left: 50,
			right: 50,
			top: 50,
			bottom: 30
		};

		width = width - margin.left - margin.right;
		height = height - margin.top - margin.bottom;

		let interestingKeysMost = [
			'concave_points_worst',
			'perimeter_worst',
			'concave_points_mean',
			'radius_worst',
			'perimeter_mean'
		];

		let interestingKeysLeast = [
			'smoothness_se',
			'fractal_dimension_mean',
			'texture_se',
			'symmetry_se',
			'fractal_dimension_se'
		];

		// concat the two arrays into keys
		let keys = interestingKeysMost.concat(interestingKeysLeast);

		// for all keys except 'id' and 'diagnosis' draw a density plot
		// with the corresponding data
		// id and diagnosis are the first two keys

		for (let i = 0; i < keys.length; i++) {
			let key = keys[i];
			let min_value = d3.min(dataOriginal, (d: any) => d[key] as number) as number;
			let max_value = d3.max(dataOriginal, (d: any) => d[key] as number) as number;
			let xScale = d3.scaleLinear().domain([min_value, max_value]).range([0, width]);

			let histogram = d3
				.histogram()
				.value((d: any) => d[key])
				.domain([min_value, max_value])
				.thresholds(xScale.ticks(50));

			let stdBin = d3.deviation(dataOriginal, (d: any) => d[key]) as number;

			let kde = kernelDensityEstimator(kernelEpanechnikov(stdBin / 3), xScale.ticks(50));
			let densityBegnin = kde(
				dataOriginal.filter((d: any) => d.diagnosis == 0).map((d: any) => d[key])
			);
			let densityMalignant = kde(
				dataOriginal.filter((d: any) => d.diagnosis == 1).map((d: any) => d[key])
			);

			densityBegnin[0][1] = 0;
			densityBegnin[densityBegnin.length - 1][1] = 0;

			densityMalignant[0][1] = 0;
			densityMalignant[densityMalignant.length - 1][1] = 0;

			let maxDensityBenign = d3.max(densityBegnin, (d: any) => d[1]);
			let maxDensityMalignant = d3.max(densityMalignant, (d: any) => d[1]);
			//@ts-ignore
			let maxDensity = Math.max(maxDensityBenign, maxDensityMalignant);

			let yScaleDensity = d3.scaleLinear().domain([0, maxDensity]).range([height, 0]);

			let binsBegnin = histogram(dataOriginal.filter((d: any) => d.diagnosis == 0));
			let binsMalignant = histogram(dataOriginal.filter((d: any) => d.diagnosis == 1));

			let maxLen = Math.max(
				d3.max(binsBegnin, (d: any) => d.length),
				d3.max(binsMalignant, (d: any) => d.length)
			);

			let yScale = d3.scaleLinear().range([height, 0]).domain([0, maxLen]);

			// append new div for each feature

			// check if key is in interestingKeysMost or interestingKeysLeast
			// and append to the corresponding div
			let correctDiv = null;

			if (interestingKeysMost.includes(key)) {
				correctDiv = divFeaturesMost;
			} else if (interestingKeysLeast.includes(key)) {
				correctDiv = divFeaturesLeast;
			} else {
				continue;
			}

			d3.select(correctDiv)
				.append('div')
				.attr('id', key)
				.attr('class', 'flex justify-center flex-col lg:flex-row items-center gap-5');

			// select the last div appended
			// and append a new svg
			let currentDiv = d3.select(`#${key}`).node() as HTMLDivElement;

			let keyDisplay = key.replaceAll('_', ' ');

			d3.select(currentDiv)
				.append('h1')
				.text(keyDisplay)
				.attr('class', 'font-display text-2xl font-bold text-center');

			let svg = d3
				.select(currentDiv)
				.append('svg')
				.attr('width', width + margin.left + margin.right)
				.attr('height', height + margin.top + margin.bottom)
				.append('g')
				.attr('transform', `translate(${margin.left}, ${margin.top})`);

			// X axis
			svg
				.append('g')
				.attr('id', `${key}-x_axis`)
				.attr('transform', `translate(0, ${height})`)
				.call(d3.axisBottom(xScale));

			// Y axis
			svg.append('g').attr('id', `${key}-y_axis`).call(d3.axisLeft(yScale));

			// kde for benign
			svg
				.append('g')
				.attr('id', `${key}-density-begnin`)
				.append('path')
				.datum(densityBegnin)
				.attr('class', 'fill-primary')
				.attr('opacity', '.2')
				.attr('stroke', '#000')
				.attr('stroke-width', 1)
				.attr('stroke-linejoin', 'round')
				.attr(
					'd',
					d3
						.line()
						.curve(d3.curveBasis)
						.x(function (d) {
							return xScale(d[0]);
						})
						.y(function (d) {
							return yScaleDensity(d[1]);
						})
				);

			// kde for malignant
			svg
				.append('g')
				.attr('id', `${key}-density-malignant`)
				.append('path')
				.datum(densityMalignant)
				.attr('class', 'fill-error')
				.attr('opacity', '.2')
				.attr('stroke', '#000')
				.attr('stroke-width', 1)
				.attr('stroke-linejoin', 'round')
				.attr(
					'd',
					d3
						.line()
						.curve(d3.curveBasis)
						.x(function (d) {
							return xScale(d[0]);
						})
						.y(function (d) {
							return yScaleDensity(d[1]);
						})
				);
			// append the bar rectangles to the svg element
			let rectBegnin = svg
				.append('g')
				.attr('id', `${key}-begnin`)
				.selectAll('rect')
				.data(binsBegnin)
				.enter()
				.append('rect')
				.attr('x', 1)
				.attr('transform', (d: any) => `translate(${xScale(d.x0)}, ${yScale(d.length)})`)
				.attr('width', (d: any) => xScale(d.x1) - xScale(d.x0) - 1)
				.attr('height', (d: any) => height - yScale(d.length))
				.attr('class', 'fill-primary')
				.attr('fill-opacity', 0.5);

			let rectMalignant = svg
				.append('g')
				.attr('id', `${key}-malignant`)
				.selectAll('rect')
				.data(binsMalignant)
				.enter()
				.append('rect')
				.attr('x', 1)
				.attr('transform', (d: any) => `translate(${xScale(d.x0)}, ${yScale(d.length)})`)
				.attr('width', (d: any) => xScale(d.x1) - xScale(d.x0) - 1)
				.attr('height', (d: any) => height - yScale(d.length))
				.attr('class', 'fill-error')
				.attr('fill-opacity', 0.5);

			// add legends for Being and Malignant
			// circles on top right corner with the text

			svg
				.append('g')
				.attr('id', `${key}-legend`)
				.append('circle')
				.attr('cx', width - margin.right * 3)
				.attr('cy', 0)
				.attr('r', 5)
				.attr('class', 'fill-primary');

			svg
				.select(`#${key}-legend`)
				.append('text')
				.attr('x', width - margin.right * 3 + 10)
				.attr('y', 5)
				.text('Begnin')
				.attr('class', 'fill-current');

			svg
				.select(`#${key}-legend`)
				.append('circle')
				.attr('cx', width - margin.right * 3)
				.attr('cy', margin.top / 2)
				.attr('r', 5)
				.attr('class', 'fill-error');

			svg
				.select(`#${key}-legend`)
				.append('text')
				.attr('x', width - margin.right * 3 + 10)
				.attr('y', margin.top / 2 + 5)
				.text('Malignant')
				.attr('class', 'fill-current');

			// add hover effect

			rectBegnin
				.on('mouseover', function (event: any, d: any) {
					d3.select(this).attr('fill-opacity', 1);
					let rectWithSameX = rectMalignant.filter((d2: any) => d2.x0 == d.x0);
					rectWithSameX.attr('fill-opacity', 0.2);

					// add border to the corresponding density plot
					svg.select(`#${key}-density-begnin`).select('path').attr('stroke-width', 3);

					svg
						.append('text')
						.attr('x', xScale(d.x0) + 10)
						.attr('y', yScale(d.length) - 10)
						.text(d.length)
						.attr('class', 'fill-current')
						.attr('id', 'tooltip');
				})
				.on('mouseout', function (event: any, d: any) {
					d3.select(this).attr('fill-opacity', 0.5);
					let rectWithSameX = rectMalignant.filter((d2: any) => d2.x0 == d.x0);
					rectWithSameX.attr('fill-opacity', 0.5);

					svg.select(`#${key}-density-begnin`).select('path').attr('stroke-width', 1);

					svg.select('#tooltip').remove();
				});

			rectMalignant
				.on('mouseover', function (event: any, d: any) {
					d3.select(this).attr('fill-opacity', 1);
					let rectWithSameX = rectBegnin.filter((d2: any) => d2.x0 == d.x0);
					rectWithSameX.attr('fill-opacity', 0.2);
					// add border to the corresponding density plot
					svg.select(`#${key}-density-malignant`).select('path').attr('stroke-width', 3);
					svg
						.append('text')
						.attr('x', xScale(d.x0) + 10)
						.attr('y', yScale(d.length) - 10)
						.text(d.length)
						.attr('class', 'fill-current')
						.attr('id', 'tooltip');
				})
				.on('mouseout', function (event: any, d: any) {
					d3.select(this).attr('fill-opacity', 0.5);
					let rectWithSameX = rectBegnin.filter((d2: any) => d2.x0 == d.x0);
					rectWithSameX.attr('fill-opacity', 0.5);
					svg.select(`#${key}-density-malignant`).select('path').attr('stroke-width', 1);
					svg.select('#tooltip').remove();
				});
		}
	}
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
		let circles = svg
			.append('g')
			.selectAll('circle')
			.data(dataPCA)
			.enter()
			.append('circle')
			.attr('cx', (d: any) => xScale(d.PC1))
			.attr('cy', (d: any) => yScale(d.PC2))
			.attr('r', 5)
			.attr('class', (d: any) => (d.diagnosis == 0 ? 'fill-primary ' : 'fill-error'))
			.attr('fill-opacity', 0.3);

		// add legend text on the right using a small circle and a text
		circles
			.on('mouseover', function (event: any, d: any) {
				let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);

				circlesWithSameDiagnostic
					.attr('fill-opacity', 1)
					.attr('stroke', 'black')
					.attr('stroke-width', 1);

				d3.select(this).attr('r', 10);

				svg
					.append('text')
					.attr('x', xScale(d.PC1) + 10)
					.attr('y', yScale(d.PC2) - 10)
					.text(d.id)
					.attr('class', 'fill-current')
					.attr('id', 'tooltip');
			})
			.on('mouseout', function (event: any, d: any) {
				let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);

				circlesWithSameDiagnostic.attr('fill-opacity', 0.3).attr('stroke', 'none');
				d3.select(this).attr('r', 5);
				svg.select('#tooltip').remove();
			});

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
		let circles = svg
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

		circles.on('mouseover', function (event: any, d: any) {
			let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);

			circlesWithSameDiagnostic
				.attr('fill-opacity', 1)
				.attr('stroke', 'black')
				.attr('stroke-width', 1);

			d3.select(this).attr('r', 10);

			svg
				.append('text')
				.attr('x', xScale(d.x) + 10)
				.attr('y', yScale(d.y) - 10)
				.text(d.id)
				.attr('class', 'fill-current')
				.attr('id', 'tooltip');
		});

		circles.on('mouseout', function (event: any, d: any) {
			let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);
			circlesWithSameDiagnostic.attr('fill-opacity', 0.3).attr('stroke', 'none');
			d3.select(this).attr('r', 5);
			svg.select('#tooltip').remove();
		});

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
			.range([margin + height * 0.15, height * 0.85 - margin])
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
		let rectangles = svg
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

		rectangles.on('mouseover', function (event: any, d: any) {
			svg
				.append('text')
				.attr('x', (xScale(d.j) as number) + xScale.bandwidth() / 2)
				.attr('y', (yScale(d.i) as number) + yScale.bandwidth() / 2)
				.text(d.value.toFixed(2))
				.attr('class', 'fill-current')
				.attr('id', 'tooltip');
		});

		rectangles.on('mouseout', function (event: any, d: any) {
			d3.select('#tooltip').remove();
		});

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

		let legendScale = d3
			.scaleLinear()
			.domain([min_value, max_value])
			.range([width * 0.4, width * 0.9]);
		// add legend with rectangles of colors and text
		svg
			.append('g')
			.attr('id', 'legend')
			.selectAll('rect')
			.data(d3.range(min_value, max_value, (max_value - min_value) / 100))
			.enter()
			.append('rect')
			.attr('x', (d, i) => legendScale(d) as number)
			.attr('y', height * 0.11)
			.attr('width', (d: any) => legendScale(d + (max_value - min_value) / 100) - legendScale(d)) // fit up to the next rect
			.attr('height', yScale.bandwidth())
			.style('fill', (d: any) => colorScale(d));

		// add legend scale with ticks and make it reach the end
		svg
			.append('g')
			.attr('id', 'legend-scale')
			.attr('transform', `translate(0, ${height * 0.11 + yScale.bandwidth()})`)
			.call(
				d3
					.axisBottom(legendScale)
					.tickValues(d3.range(min_value, max_value * 1.1, (max_value - min_value) / 10))
			)
			.selectAll('text')
			.attr('class', 'fill-current');

		// Add legend title
		svg
			.append('text')
			.attr('x', margin + width * 0.22)
			.attr('y', height * 0.11 + yScale.bandwidth())
			.text('Correlation scale :')
			.attr('class', 'fill-current md:text-xl text-xs')
			.style('text-anchor', 'middle');
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
				<div class="md:flex block justify-center items-center mdv:mt-3 mb-6">
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

		<div>
			<h1 class="mx-auto text-center text-3xl mb-10">Top 5 most correlated features</h1>
			<div
				class="md:mx-10 mx-2 flex flex-col justify-center items-center gap-10"
				bind:this={divFeaturesMost}
			></div>
			<h1 class="mx-auto text-center text-3xl my-8">Top 5 least correlated features</h1>
			<div
				class="md:mx-10 mx-2 flex flex-col justify-center items-center gap-10"
				bind:this={divFeaturesLeast}
			></div>
		</div>

		<div class="divider m-10" />
	</div>
{/if}
