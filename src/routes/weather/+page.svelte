<script lang="ts">
	import Countries from '$lib/Countries.svelte';
	import Cities from '$lib/Cities.svelte';
	import CityWeather from '$lib/CityWeather.svelte';
	import weatherStore from '$stores/weatherStore';
	import citiesStore from '$stores/citiesStore';
	import SEO from '$lib/SEO.svelte';
	import Loader from '$lib/Loader.svelte';
	export let data;
	let loading = false;
	$: ({ weather } = data);
	let cityWeather = {};
	const getCities = async (selectedCountry: any) => {
		let data;
		try {
			const res = await fetch(`/api/${selectedCountry}.cities.json`);
			data = await res.json();
		} catch (e) {
			console.log(e);
		}
		return data;
	};

	const getCityWeather = async (selectedCity: any) => {
		let data;
		try {
			const res = await fetch(`/api/${selectedCity}.weather.json`);
			data = await res.json();
		} catch (e) {
			console.log(e);
		}
		return data;
	};
	$: console.log($citiesStore);
	let cities;
	$: cities = $citiesStore?.cities ? $citiesStore?.cities : [];
</script>

<SEO title={'City Weather'} description={'World City Weather'} />

<div class="flex justify-center align-middle items-center">
	<div
		class="p-4 m-2 border border-gray-200 dark:border-gray-500 rounded shadow bg-white dark:bg-gray-600"
	>
		<h1 class="p-2 font-bold text-gray-700 dark:text-gray-200">City Weather</h1>

		<div class="flex flex-wrap gap-2">
			<Countries
				data={weather}
				on:selectedCountry={async (e) => {
					if (e.detail === $citiesStore?.countryCode) {
						return;
					}
					citiesStore.set({ countryCode: e.detail, cities: null });
					weatherStore.update((state) => {
						state.loading = false;
						state.result = null;
						return state;
					});
					loading = true;
					let data = await getCities(e.detail);
					citiesStore.set({ countryCode: e.detail, cities: data.cities });
					loading = false;
				}}
			/>
			<div class="flex items-center ">
				<div class="relative">
					<Cities
						data={cities}
						on:selectedCity={async (e) => {
							weatherStore.update((state) => {
								state.loading = true;
								return state;
							});
							cityWeather = await getCityWeather(e.detail);
							weatherStore.update((state) => {
								state.loading = false;
								state.result = cityWeather;
								return state;
							});
						}}
					/>
					{#if loading}<div class="absolute right-1 top-7"><Loader /></div>{/if}
				</div>
			</div>
		</div>
		<div class="flex">
			<CityWeather />
		</div>
	</div>
</div>
