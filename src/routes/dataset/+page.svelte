<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	let loaded = false;

	onMount(() => {
		loaded = true; // this is for the fade transition
	});

	let features = [
		{
			title: 'Smoothness',
			description: 'quantifies the difference in length between a given radius and its neighbors'
		},
		{
			title: 'Concavity',
			description:
				'measures the number and severity of the concavities observed in the cells (by drawing chords all around the cell border)'
		},
		{
			title: 'Concave Points',
			description: 'only record the number of such concavities'
		},
		{
			title: 'Symmetry',
			description:
				'quantifies the difference in length between symmetrical perpendicular lines located on the preferred orientation axis'
		},
		{
			title: 'Fractal Dimension',
			description:
				'Using the coastline approximation, researchers measured an approximation of the cell’s perimeter with respect to the length of inside rulers. As the perimeter decreases when the ruler length increases, the computed negative slope of the log-scaled values in this relationship is the fractal dimension of the cell'
		},
		{
			title: 'Texture',
			description:
				'represents the standard deviation of the cell pixel values when converted in grayscale'
		}
	];
</script>

<svelte:head>
	<title>Dataset</title>
</svelte:head>

{#if loaded}
	<div in:fade={{ duration: 400, delay: 100 }} class="mx-auto transition duration-500">
		<div class="bg-base-100 py-10 px-4 mx-auto mb-10">
			<div class="flex flex-col items-center justify-stretch group mx-auto overflow-visible">
				<div class="md:flex block justify-center items-center md:mt-3 mb-6">
					<h1
						class="text-center font-display text-5xl md:text-7xl font-bold bg-clip-text bg-secondary anim-gradient pb-3"
					>
						Dataset
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center">Breast Cancer Wisconsin Dataset</div>
				</div>
				<div class="text-center md:max-w-3xl">
					In details presentation of the dataset features : Radius, Perimeter, Area, Compactness,
					Smoothness, Concavity, Concave Points, Symmetry, Fractal Dimension, Texture
				</div>
			</div>
		</div>

		<div class="divider m-10" />

		<div class="mx-10 flex flex-col justify-start gap-5">
			<div>
				<h1 class="text-left w-full font-bold text-xl mb-2">Features Presentation</h1>
				<div class="text-justify">
					The dataset describes 569 microscopic images of breast cells which were collected on
					patients for diagnosis. As each image might have included multiple sets, all features are
					computed for each cells and stored as average, standard error and worst (mean of the three
					largest values) giving a total of 30 features on top of the id, and diagnosis (Malignant
					or Benign).
				</div>
			</div>

			<div class="flex justify-start flex-col md:flex-row items-center">
				<h1 class="text-left basis-1/6 font-bold text-lg mr-2">Radius</h1>
				<div class="text-justify basis-5/6">
					calculated as the mean distance between a cell’s borders and its computed centroid
				</div>
			</div>
			<div class="flex justify-start flex-col md:flex-row items-center">
				<h1 class="text-left basis-1/6 font-bold text-lg mr-2">Perimeter</h1>
				<div class="text-justify basis-5/6">is defined as the length of the cell borde</div>
			</div>
			<div class="flex justify-start flex-col md:flex-row items-center">
				<h1 class="text-left basis-1/6 font-bold text-lg mr-2">Area</h1>
				<div class="text-justify basis-5/6">
					is the number of pixels inside the border, plus one-half of the border pixels
				</div>
			</div>

			<div class="flex justify-start flex-col md:flex-row items-center">
				<h1 class="text-left basis-1/6 font-bold text-lg mr-2">Compactness</h1>
				<div class="text-justify grow fill-current stroke-red flex items-center">
					<p>grasps the cell irregularity computing</p>
					<svg
						version="1.1"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 381.0901412002697 129.1246974608075"
						width="100"
						height="50"
						class="ml-2"
					>
						<defs>
							<style class="style-fonts">
								@font-face {
									font-family: 'Virgil';
									src: url('https://excalidraw.com/Virgil.woff2');
								}
								@font-face {
									font-family: 'Cascadia';
									src: url('https://excalidraw.com/Cascadia.woff2');
								}
								@font-face {
									font-family: 'Assistant';
									src: url('https://excalidraw.com/Assistant-Regular.woff2');
								}
							</style>
						</defs>
						<rect x="0" y="0" width="381.0901412002697" height="129.1246974608075" fill="none"
						></rect><g transform="translate(55.993511342422835 10) rotate(0 86.98332977294922 22.5)"
							><text
								x="0"
								y="0"
								font-family="Virgil, Segoe UI Emoji"
								font-size="36px"
								fill="currentColor"
								text-anchor="start"
								style="white-space: pre;"
								direction="ltr"
								dominant-baseline="text-before-edge">perimeter²</text
							></g
						><g stroke-linecap="round"
							><g
								transform="translate(11.462471439084084 63.43724788400641) rotate(0 125.57753252741509 -1.781241596133441)"
								><path
									d="M0 0.11 C42.08 -0.31, 210.23 -3.24, 252.28 -3.7 M-1.46 -0.88 C40.58 -1.01, 209.66 -1.63, 251.92 -2.24"
									stroke="currentColor"
									stroke-width="2"
									fill="none"
								></path></g
							></g
						><mask></mask><g
							transform="translate(95.18082645736081 74.12469746080751) rotate(0 41.58333206176758 22.5)"
							><text
								x="0"
								y="0"
								font-family="Virgil, Segoe UI Emoji"
								font-size="36px"
								fill="currentColor"
								text-anchor="start"
								style="white-space: pre;"
								direction="ltr"
								dominant-baseline="text-before-edge">area</text
							></g
						><g
							transform="translate(296.4611268204519 20.635838791009064) rotate(0 37.31450718990891 39.44760055616484)"
							><text
								x="0"
								y="0"
								font-family="Virgil, Segoe UI Emoji"
								font-size="63.11616088986363px"
								fill="currentColor"
								text-anchor="start"
								style="white-space: pre;"
								direction="ltr"
								dominant-baseline="text-before-edge">- 1</text
							></g
						></svg
					>
				</div>
			</div>

			{#each features as { title, description }}
				<div class="flex justify-start flex-col md:flex-row items-center">
					<h1 class="text-left basis-1/6 font-bold text-lg mr-2">{title}</h1>
					<div class="text-justify basis-5/6">{description}</div>
				</div>
			{/each}
		</div>

		<div class="divider m-10" />
	</div>
{/if}
