<script>
    import MyMap from "./MyMap.svelte";
    import {csvParse} from 'd3-dsv'
	import postcodes from './postcodes'
	import Switch from './Switch.svelte'
  import Timeseries from "./Timeseries.svelte";
  import RangeSlider from "svelte-range-slider-pips";
//create GitHub repo
//upload data
//https://cdn.statically.io/gh/ONSvisual/local-statistics/dev/static/data/places.csv

	let data, timePeriod='2023Q3', from="cardholder", to="merchant", pca="SO40",sliderValue;

	let periods=['2019Q1', '2019Q2', '2019Q3', '2019Q4', '2020Q1', '2020Q2', '2020Q3', '2020Q4', '2021Q1', '2021Q2', '2021Q3', '2021Q4', '2022Q1', '2022Q2', '2022Q3', '2022Q4', '2023Q1', '2023Q2', '2023Q3']
function toggleFlow(sliderValue){
	if(sliderValue=="off"){from="merchant"; to="cardholder"}
	if(sliderValue=="on"){from="cardholder"; to="merchant"}
	from=from;
	to=to;
}

let postcode, click;
$: toggleFlow(sliderValue)

let keys=postcodes;
let decimals = 2;

</script>

<div>{decimals} decimal places</div>
  <label>
    <input bind:group={decimals} type="radio" name="amount" value=0 /> 0
  </label>
  <label>
    <input bind:group={decimals} type="radio" name="amount" value=1 /> 1
  </label>
  <label>
    <input bind:group={decimals} type="radio" name="amount" value=2 /> 2
  </label>




<div class="cont">
    <label class="label" for="#selector0">Postal area</label>
    <select id="selector0" bind:value={pca} on:change={()=>click=1}>
      {#each keys as s, i}
        <option class="option" value={s}>{s}</option>
      {/each}
    </select>
  </div>

<Switch bind:value={sliderValue} label="merchants in {pca}" label2="cardholders in {pca}" fontSize={18} design="slider" />
<p>
{#if sliderValue=="on"}
Cardholders resident in {pca}: where they spent their money in Q{timePeriod.split("Q")[1]}-{timePeriod.split("Q")[0]}
{/if}
{#if sliderValue=="off"}
Merchants based in {pca}: where their customers came from in {timePeriod}
{/if}
</p>
<RangeSlider 
values={[periods.length]} 
min=0 
max={periods.length-1} 
pips 
float={true} 
pipstep=1 
all='label'
formatter={(v)=>periods[v].replace("20","'").replace("Q","<br>Q")}
handleFormatter={(v)=>periods[v].replace("20","'").replace("Q","<br>Q")}
id="slider"
on:change={(e) => timePeriod=periods[e.detail.value]}
springValues={ {stiffness: 0.9, damping: 0.9} }
/>

<!-- {#if $selected}
<Timeseries
  {w}
  {data}
  {selected}
  {date}
  {options}
  series={data.filter(
    (e) =>
      e.source == $selected.filter0 && e.dest == $selected.filter1
  )}
   transform=null
/>
{/if} -->

<MyMap  {timePeriod} {from} {to} {pca} bind:selected={pca} {decimals}></MyMap>

<style>
:global(#slider){margin-bottom:4.5em;}

select {
    -moz-appearance: none; /* Firefox */
    -webkit-appearance: none; /* Safari and Chrome */
    max-width: calc(100%) !important;
    appearance: none;
    position: relative;
    display: inline-block;
    vertical-align: middle;
    font-size: 14px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    border-radius: 0;
    background-color: transparent;
    background: url("../images/downArrow.svg");
    background-position: center;
    background-size: 30px 30px;
    background-position-x: 30px;
    background-position-y: 5px;
    background-position: right;
    background-repeat: no-repeat;
    border-radius: none;
    border: 2px solid #206095;
    margin: 0;
  }
  :global(body){   
	max-width: 800px;
}
</style>