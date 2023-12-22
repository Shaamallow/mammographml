<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

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
			deathData = await preprocess_death_data('/map/breast_cancer_deaths.csv');
			worldMap = await d3.json('/map/worldMap.geojson');
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
	<div in:fade={{ duration: 400, delay: 100 }} class="mx-auto transition duration-500">
		<div class="bg-base-100 py-10 px-4 mx-auto mb-10">
			<div
				class="flex flex-col items-center justify-stretch group max-w-3xl mx-auto overflow-visible"
			>
				<div class="md:flex block justify-center items-center md:mt-3 mb-6">
					<h1
						class="text-center font-display text-5xl md:text-7xl font-bold bg-clip-text bg-secondary anim-gradient pb-3"
					>
						Mammograph ML
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center italic">
						Machine Learning and Features analysis on breast cancer cells observations
					</div>
				</div>
				<div class="text-center max-w-md">
					Using the breast Cancer Wisconsin Dataset, we analyzed 33 features on from 569
					observations of tumors describing the characteristics of their nuclei cells.
				</div>
			</div>
		</div>

		<!-- Card for cells -->
		<div class="mx-auto p-2 md:max-w-[60rem]">
			<div class="card bg-neutral shadow-xl">
				<figure>
					<img src="/cells_wallpaper.jpg" alt="cell" />
				</figure>
				<div class="card-body">
					<h2 class="card-title">World Global disease</h2>
					<p class="text-justify">
						Breast cancer is characterized by the uncontrolled growth of certain cell types in the
						breast tissue. Cells may eventually spread out in other parts of the body resulting in
						deadly metastasis. In 2020, <strong>2.3</strong> million women were diagnosed with
						breast cancer, and <strong>685,000 died</strong> from this condition globally. It is the
						most prevalent cancer in the world. Roughly half of all breast cancers occur in women with
						no specific risk factor other than age and sex. Men can also suffer from this condition,
						around 0.5-1%
					</p>
					<div class="card-actions justify-end">
						<input
							type="button"
							value="See More"
							class="btn btn-error"
							on:click={() => {
								window.scrollTo({
									//@ts-ignore
									top: document.getElementById('map-div').offsetTop,
									behavior: 'smooth'
								});
							}}
						/>
					</div>
				</div>
			</div>
		</div>

		<div class="m-10 mb-20 flex justify-center flex-col">
			<h1
				class="font-display text-xl md:text-3xl font-bold bg-clip-text bg-secondary mb-3 text-center"
			>
				Statistics
			</h1>
			<div class="flex justify-center">
				<div class="stats shadow bg-neutral">
					<div class="stat">
						<div class="stat-figure text-error">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="icon icon-tabler icon-tabler-grave-2"
								width="44"
								height="44"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								fill="none"
								stroke-linecap="round"
								stroke-linejoin="round"
							>
								<path stroke="none" d="M0 0h24v24H0z" fill="none" />
								<path d="M7 16.17v-9.17a3 3 0 0 1 3 -3h4a3 3 0 0 1 3 3v9.171" />
								<path d="M12 7v5" />
								<path d="M10 9h4" />
								<path d="M5 21v-2a3 3 0 0 1 3 -3h8a3 3 0 0 1 3 3v2h-14z" />
							</svg>
						</div>
						<div class="stat-title">Total Deaths</div>
						<div class="stat-value text-error">685,000</div>
						<div class="stat-desc">in 2020</div>
					</div>

					<div class="stat">
						<div class="stat-figure text-secondary">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="icon icon-tabler icon-tabler-building-hospital"
								width="44"
								height="44"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								fill="none"
								stroke-linecap="round"
								stroke-linejoin="round"
							>
								<path stroke="none" d="M0 0h24v24H0z" fill="none" />
								<path d="M3 21l18 0" />
								<path d="M5 21v-16a2 2 0 0 1 2 -2h10a2 2 0 0 1 2 2v16" />
								<path d="M9 21v-4a2 2 0 0 1 2 -2h2a2 2 0 0 1 2 2v4" />
								<path d="M10 9l4 0" />
								<path d="M12 7l0 4" />
							</svg>
						</div>
						<div class="stat-title">Total Sick</div>
						<div class="stat-value text-secondary">2.3M</div>
						<div class="stat-desc">in 2020</div>
					</div>

					<div class="stat">
						<div class="stat-figure text-primary">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="icon icon-tabler icon-tabler-gender-male"
								width="44"
								height="44"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								fill="none"
								stroke-linecap="round"
								stroke-linejoin="round"
							>
								<path stroke="none" d="M0 0h24v24H0z" fill="none" />
								<path d="M10 14m-5 0a5 5 0 1 0 10 0a5 5 0 1 0 -10 0" />
								<path d="M19 5l-5.4 5.4" />
								<path d="M19 5h-5" />
								<path d="M19 5v5" />
							</svg>
						</div>
						<div class="stat-title">Risk for men</div>
						<div class="stat-value text-primary">0.5%</div>
						<div class="stat-desc">and up to 1% !</div>
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

		<div class="divider my-10" />

		<div class="w-full bg-base-100">
			<div class="flex flex-col items-center justify-stretch group max-w-3xl mx-auto">
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

				<a href="https://d3js.org/" target="_blank">
					<img
						src="https://raw.githubusercontent.com/d3/d3-logo/master/d3.png"
						alt="d3"
						class="mt-3 w-20"
					/></a
				>
			</div>
		</div>

		<!-- PROJECTS FROM HERE -->
		<div class="divider m-10" />
	</div>
{/if}
