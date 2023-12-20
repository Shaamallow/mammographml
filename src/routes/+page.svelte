<script lang="ts">
	import type { PageData } from './$types';

	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	let loaded = false;
	let map_svg: SVGElement;

	onMount(() => {
		loaded = true;
	});
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

		<div class="divider my-20" id="map-div" />

		<div class="w-11/12 rounded mx-auto bg-neutral h-[50rem]" id="map">
			<h1
				class="font-display text-xl md:text-3xl font-bold bg-clip-text bg-secondary mb-3 text-center"
			>
				La map ici
			</h1>
			<svg bind:this={map_svg}></svg>
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

<style>
	.anim-gradient {
		background-image: linear-gradient(
			300deg,
			rgb(var(--ctp-red)),
			rgb(var(--ctp-peach)),
			rgb(var(--ctp-yellow)),
			rgb(var(--ctp-green)),
			rgb(var(--ctp-blue)),
			rgb(var(--ctp-mauve))
		);

		background-size: 360% 360%;
		animation: gradient-animation 10s ease infinite;
	}

	/*Keyframes*/
	@keyframes gradient-animation {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}
</style>
