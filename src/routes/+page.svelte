<script>

  import {onMount} from 'svelte'
  import {csv, autoType} from 'd3'
  import Select from '../components/Select.svelte';
  import LineChart from '../components/LineChart.svelte';


  let data = []
  let selectedValue = 'personnel'

	const getDayOfYear = (date) => {
	  const startOfYear = new Date(date.getFullYear(), 0, 1);
		const diffInMillis = date - startOfYear;
		return Math.floor(diffInMillis / (1000 * 60 * 60 * 24)) + 1;
	}

	onMount(async function() {
		data = await csv(
			'https://raw.githubusercontent.com/romansverdan/Russian-Losses-in-Ukraine/main/data/russia_losses.csv', 
			autoType
		);
	});

	$: dataWithDays = data.map(el => {
		return {
			...el,
			dayNumber: getDayOfYear(el.date)
		}
	})

	let categories = ['personnel']
	$: if(data[0]) {
		categories = Object.keys(data[0]).slice(2)
	} 
	
</script>

<h1>Yearly Russian Combat Losses in Ukraine since 2/22</h1>

{#if data.length > 0}
  <Select bind:selectedValue items={categories}/>
  <LineChart 
    data={dataWithDays}
    category={selectedValue}
    yLabel="Total losses"
  />
{/if}

<p>Data source: <a href="https://x.com/GeneralStaffUA" target="_blank">General Staff of Ukraine</a></p>

<p>More charts of same dataset <a href="https://observablehq.com/collection/@romansverdan/russian-war-data" target="_blank">here</a></p>


<style>
  p {
    font-size: 14px;
  }
</style>