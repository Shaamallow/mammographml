<script lang="ts">
	import type { PageData } from './$types';
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	export let data: PageData;

	let loaded = false;
	let mindmap_svg: SVGElement;

	onMount(() => {
		loaded = true;
	});
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
						Dataset
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center">Breast Cancer Wisconsin Dataset</div>
				</div>
				<div class="text-center max-w-md">
					More information about the dataset here : Features descriptions, cout, and representations
					available
				</div>
			</div>
		</div>

		<!-- MIND MAP ICIs -->

		<div class="w-11/12 rounded mx-auto bg-neutral h-[50rem]" id="mindmap">
			<h1
				class="font-display text-xl md:text-3xl font-bold bg-clip-text bg-secondary mb-3 text-center"
			>
				MIND MAP
			</h1>
			<svg bind:this={mindmap_svg}></svg>
		</div>

		<div class="divider my-20" />

		<div class="mx-10 flex flex-wrap justify-center items-center gap-10">
			<div class="w-[50rem] bg-red-100 h-[20rem]">Feature</div>
			<div class="w-[50rem] bg-red-100 h-[20rem]">Feature</div>
			<div class="w-[50rem] bg-red-100 h-[20rem]">Feature</div>
			<div class="w-[50rem] bg-red-100 h-[20rem]">Feature</div>
		</div>

		<!-- PROJECTS FROM HERE -->
		<div class="divider m-10" />
	</div>
{/if}
