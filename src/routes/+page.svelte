<script lang="ts">
	import type { PageData } from './$types';

	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import { geoNaturalEarth1 } from 'd3-geo';

	let loaded = false;
	let divMap: HTMLElement;

	let deathData: any;
	let worldMap: any;

	onMount(() => {
		loaded = true;

		async function preprocess_death_data(path: string) {
			let data = await d3.csv(path);
			let data_preprocessed: any;
			data_preprocessed = [];
			for (let i = 0; i < data.length; i++) {
				if (data[i]['Year'] == '2016') {
					data_preprocessed.push({
						Country: data[i]['Country'],
						Deaths: parseFloat(data[i]['Deaths'])
					});
				}
			}
			data_preprocessed.push({
				Country: 'Republic of the Congo',
				Deaths: parseFloat(data_preprocessed[47]['Deaths'])
			});
			data_preprocessed.push({
				Country: 'Democratic Republic of the Congo',
				Deaths: parseFloat(data_preprocessed[54]['Deaths'])
			});
			data_preprocessed.push({
				Country: 'Guinea Bissau',
				Deaths: parseFloat(data_preprocessed[84]['Deaths'])
			});
			data_preprocessed.push({
				Country: 'Republic of Serbia',
				Deaths: parseFloat(data_preprocessed[168]['Deaths'])
			});
			data_preprocessed.push({
				Country: 'United Republic of Tanzania',
				Deaths: parseFloat(data_preprocessed[194]['Deaths'])
			});
			data_preprocessed.push({
				Country: 'USA',
				Deaths: parseFloat(data_preprocessed[208]['Deaths'])
			});

			return data_preprocessed;
		}

		const loadData = async () => {
			deathData = await preprocess_death_data('/breast_cancer_deaths.csv');
			worldMap = await d3.json('/worldMap.geojson');
		};

		loadData().then(() => {
			drawDeathMap(deathData);
		});

		window.addEventListener('resize', () => {
			drawDeathMap(deathData);
		});
	});

	function drawDeathMap(data: any) {
		divMap.innerHTML = '';

		let width = divMap.clientWidth;
		let height = width / 2;

		let margin = 20;

		const svg = d3
			.select(divMap)
			.append('svg')
			.attr('width', width)
			.attr('height', height + 50);

		let projection = d3
			.geoNaturalEarth1()
			.scale(width / 2 / Math.PI)
			.translate([width / 2, height / 2 + 50]);

		let path: any = d3.geoPath().projection(projection);

		console.log(data);
		// china is max, should remove World, Western Europe and all that are not real countries...
		let maxValue: number = data.find((d: any) => d.Country == 'China').Deaths as number;
		let minValue: number = d3.min(data, (d: any) => d.Deaths as number) as number;
		maxValue = Math.ceil(maxValue);

		let colorScaleWorld = d3
			.scaleSequential(d3.interpolateReds)
			.domain([Math.log(minValue / 2), Math.log(maxValue / 2)]);

		let countries = svg
			.selectAll('path')
			.data(worldMap.features)
			.enter()
			.append('path')
			.attr('d', path)
			.attr('fill', (d: any) => {
				const countryData = data.find((country: any) => country.Country == d.properties.name);
				return countryData ? colorScaleWorld(Math.log(countryData.Deaths / 2)) : '#ccc';
			})
			.attr('stroke', '#fff')
			.attr('stroke-width', 0.5);

		countries.on('mouseover', (event: any, d: any) => {
			d3.select(event.currentTarget).attr('stroke-width', 2);

			// compute centroid of path
			let centroid = path.centroid(d);
			// add tooltip at mouse position with country name and deaths

			// get country data
			let countryData = data.find((country: any) => country.Country == d.properties.name);
			let deathString = countryData ? countryData.Deaths.toFixed(0) : 'No data';

			svg
				.append('text')
				.attr('id', 'tooltip_name')
				.attr('x', centroid[0])
				.attr('y', centroid[1])
				.attr('text-anchor', 'middle')
				.attr('class', 'md:text-2xl text-sm')
				.attr('font-weight', 'bold')
				.attr('fill', 'black')
				.text(`${d.properties.name}`);

			svg
				.append('text')
				.attr('id', 'tooltip_deaths')
				.attr('x', centroid[0])
				.attr('y', centroid[1] + 20)
				.attr('text-anchor', 'middle')
				.attr('font-size', '0.8em')
				.attr('font-weight', 'bold')
				.attr('fill', 'black')
				.text(`${deathString}`);
		});

		countries.on('mouseout', (event: any, d: any) => {
			d3.select(event.currentTarget).attr('stroke-width', 0.5);

			d3.select('#tooltip_name').remove();
			d3.select('#tooltip_deaths').remove();
		});

		// add legend for color scale on top of map

		let dataLegend = d3.range(minValue, maxValue, (maxValue - minValue) / 500);
		let legendXScale = d3
			.scaleLog()
			.domain([minValue, maxValue])
			.range([margin, width - margin]);
		let legendAxis = d3.axisBottom(legendXScale).ticks(20);
		let dataLegendLog = d3.range(
			Math.log(minValue / 2),
			Math.log(maxValue / 2),
			(Math.log(maxValue / 2) - Math.log(minValue / 2)) / 500
		);

		svg.append('g').attr('transform', `translate(0, 40)`).call(legendAxis);
		svg
			.append('g')
			.attr('transform', `translate(0, 20)`)
			.attr('id', 'legendGradient')
			.selectAll('rect')
			.data(dataLegend)
			.enter()
			.append('rect')
			.attr('x', (d: any) =>
				d3
					.scaleLinear()
					.domain([minValue, maxValue])
					.range([margin, width - margin])(d)
			)
			.attr('y', 0)
			.attr('width', (width - 2 * margin) / 500)
			.attr('height', 20)
			.attr('fill', (d: any, i: any) => colorScaleWorld(dataLegendLog[i]));
	}
</script>

<svelte:head>
	<title>Cancer</title>
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
						Cancer Visualisation
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center">Breast Cancer Wisconsin Dataset</div>
				</div>
				<div class="text-center max-w-md">
					Analyze 33 features from 569 observations of tumors describing the characteristics of
					their nuclei cells.
				</div>
			</div>
		</div>

		<!-- Card for cells -->
		<div class="mx-10">
			<div class="mx-auto max-w-[50rem]">
				<div class="card md:card-side bg-neutral shadow-xl">
					<figure>
						<img src="/cells_wallpaper.jpeg" alt="cell" />
					</figure>
					<div class="card-body">
						<h2 class="card-title">le CANCER TUE</h2>
						<p>Voila des infos sur pourquoi le cancer c'est mal</p>
						<div class="card-actions justify-end">
							<a href="#map-div" class="btn btn-primary">Voir plus</a>
						</div>
					</div>
				</div>
			</div>
		</div>

		<div class="divider my-20" />

		<div class="mx-10 flex justify-center flex-col">
			<h1
				class="font-display text-xl md:text-3xl font-bold bg-clip-text bg-secondary mb-3 text-center"
			>
				Tout plein de stats la
			</h1>
			<div class="flex justify-center">
				<div class="stats shadow bg-neutral">
					<div class="stat">
						<div class="stat-figure text-primary">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
								class="inline-block w-8 h-8 stroke-current"
								><path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"
								></path></svg
							>
						</div>
						<div class="stat-title">Total Likes</div>
						<div class="stat-value text-primary">25.6K</div>
						<div class="stat-desc">21% more than last month</div>
					</div>

					<div class="stat">
						<div class="stat-figure text-secondary">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
								class="inline-block w-8 h-8 stroke-current"
								><path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M13 10V3L4 14h7v7l9-11h-7z"
								></path></svg
							>
						</div>
						<div class="stat-title">Page Views</div>
						<div class="stat-value text-secondary">2.6M</div>
						<div class="stat-desc">21% more than last month</div>
					</div>

					<div class="stat">
						<div class="stat-figure text-secondary">
							<div class="avatar online">
								<div class="w-16 rounded-full">
									<img
										src="https://media.licdn.com/dms/image/C4E03AQHb5eTI74CBAw/profile-displayphoto-shrink_800_800/0/1662392381197?e=1708560000&v=beta&t=0-R18GnswBDXGiT8C3WGNAsdshsXPOg_tzdSRqqXK9U"
										alt="test"
									/>
								</div>
							</div>
						</div>
						<div class="stat-value">86%</div>
						<div class="stat-title">Tasks done</div>
						<div class="stat-desc text-secondary">31 tasks remaining</div>
					</div>
				</div>
			</div>
		</div>

		<div class="divider my-10" id="map-div" />

		<div class="mx-auto flex flex-col justify-center items-center mb-10 p-2">
			<h1 class="text-5xl font-display font-bold text-center">Map of breast Cancer deaths</h1>

			<div class="text-center max-w-md">By Country, from 1990 to 2016</div>
		</div>

		<div class="container rounded mx-auto bg-neutral">
			<div class="w-full" bind:this={divMap}></div>
		</div>

		<div class="divider my-20" />

		<div class="w-full bg-base-100">
			<div class="flex flex-col items-center justify-stretch py-5 group max-w-3xl mx-auto">
				<h1 class="font-display text-3xl md:text-5xl font-bold bg-clip-text bg-secondary mb-3">
					More on the website
				</h1>
			</div>
		</div>
		<div class="flex flex-col items-center justify-stretch mb-30 group max-w-3xl mx-auto">
			<h1 class="font-display text-xl md:text-3xl font-bold bg-clip-text bg-secondary mb-3">
				The Tech Stack
			</h1>

			<div class="flex justify-center items-center my-3 flex-wrap gap-4 md:gap-16 mx-8">
				<a href="https://svelte.dev" target="_blank">
					<img
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Svelte_Logo.svg/199px-Svelte_Logo.svg.png"
						alt="svelte"
						class="mt-3 w-20"
					/>
				</a>
				<a href="https://www.typescriptlang.org/" target="_blank">
					<img
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Typescript_logo_2020.svg/1200px-Typescript_logo_2020.svg.png"
						alt="typescript"
						class="mt-3 w-20"
					/>
				</a>
				<a href="https://tailwindcss.com" target="_blank">
					<img
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/Tailwind_CSS_Logo.svg/512px-Tailwind_CSS_Logo.svg.png"
						alt="tailwind"
						class="mt-3 w-20"
					/>
				</a>

				<a href="https://www.python.org/" target="_blank">
					<img
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Python-logo-notext.svg/1920px-Python-logo-notext.svg.png"
						alt="python"
						class="mt-3 w-20"
					/></a
				>
			</div>
		</div>

		<!-- PROJECTS FROM HERE -->
		<div class="divider m-10" />
	</div>
{/if}
