<script>
  import MyMap from "./MyMap.svelte";
  import postcodes from "./postcodes"; //CHANGE THIS BACK !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
  import Switch from "./Switch.svelte";
  import Timeseries from "./Timeseries.svelte";
  import RangeSlider from "svelte-range-slider-pips";
  import { Radio, Radios } from "@onsvisual/svelte-components";
  import Svelecte from "svelecte";
  import { onMount } from "svelte";
  //import AutofillSelector from  "./AutofillSelector.svelte"
  let message;




  let cats = ["a", "b", "c", "d"];



  let selected;
  //create GitHub repo DONE
  //upload data (IN CHUNKS) DONE
  //https://cdn.statically.io/gh/ONSvisual/card-payment-flows/dist/data/${from}/${folder}/${file}.csv

 // user/repo/main/dist/data...

  let data,
    timePeriod = "2023Q1",
    from = "cardholder",
    to = "merchant",
    pca = "EH3",
    sliderValue;

  let periods = [
    "2019Q1",
    "2019Q2",
    "2019Q3",
    "2019Q4",
    "2020Q1",
    "2020Q2",
    "2020Q3",
    "2020Q4",
    "2021Q1",
    "2021Q2",
    "2021Q3",
    "2021Q4",
    "2022Q1",
    "2022Q2",
    "2022Q3",
    "2022Q4",
    "2023Q1",
    "2023Q2",
    "2023Q3",
  ];
  function toggleFlow(sliderValue) {
    if (sliderValue == "off") {
      from = "merchant";
      to = "cardholder";
    }
    if (sliderValue == "on") {
      from = "cardholder";
      to = "merchant";
    }
    from = from;
    to = to;
  }

  let postcode, click;
  $: toggleFlow(sliderValue);

  let keys = postcodes;
  let decimals = 2;

  let lastSelected;

  const list = postcodes;
  let temp = pca;
  let updatePca = (temp) => (temp ? (pca = temp) : (pca = pca));
  $: updatePca(temp);

  let iconValue = null;
  let merchant = true,
    cardholder = false;

  //function to be called on click
  function merchantClick() {
    merchant = false;
    cardholder = true;
    sliderValue = "on";
  }

  //function to be called on click
  function cardholderClick() {
    merchant = true;
    cardholder = false;
    sliderValue = "off";
  }

  merchantClick();

  let w;

  let processMessage = (message) => {
    if (!message) return;
    if (message.includes("pym")) return;
    else {
      if (cats.includes(message)) {
        if (message == "a") {
          timePeriod = "2023Q1";
          from = "cardholder";
          to = "merchant";
          pca = "EH3";
          sliderValue = "on"
        }
        if (message == "b") {
          timePeriod = "2023Q3";
          from = "cardholder";
          to = "merchant";
          pca = "EH3";
          sliderValue = "on"
        }
        if (message == "c") {
          timePeriod = "2019Q2";
          from = "merchant";
          to = "cardholder";
          pca = "E14";
          sliderValue = "off"
        }
        if (message == "d") {
          timePeriod = "2023Q2";
          from = "merchant";
          to = "cardholder";
          pca = "E14";
          sliderValue = "off"
        }
        timePeriod = timePeriod;
        from = from;
        to = to;
        pca = pca;
        temp = pca;
        sliderValue = sliderValue;
      }
    }
  };

  let values=[timePeriod]
  $: processMessage(message)

onMount(()=>  {
  window.addEventListener(
    "message",
    (event) => {
      //  if (!event.origin.includes("ons.gov.uk")) return;//<===================NEEDS UPDATING
      message = event.data;
    },
    false
  );})

</script>
<!-- <b><p>Notice: There will be a delay in the availability of the interactive map for the whole UK. Please check back later to explore Figure 4</p></b>NEEDS UPDATING ON PUBLISH -->
<p bind:clientWidth={w} style="width:100%"></p>
{#if w}

<label class="label" for="#selector0">Select a postal district by typing the first characters</label>
<select id="selector0" bind:value={pca} on:change={() => (click = 1)}>
  {#each keys as s, i}
    <option class="option" value={s}>{s}</option>
  {/each}
</select>

  <!-- <Svelecte
    options={list}
    bind:value={iconValue}
    clearable
    placeholder="Find a post code district"
    parentValue={postcode}
  >
    <svelte:fragment slot="clear-icon" let:selectedOptions let:inputValue
      >{@html selectedOptions.length
        ? "🗙"
        : inputValue
          ? "..."
          : '<svg viewBox="0 0 24 24" width=24 sizes="" class="Search__Image-sc-omporz-0 iPJOGI"><path fill="#141824" d="M17.121 15l5.598 5.597a.5.5 0 010 .707l-1.415 1.415a.5.5 0 01-.707 0L15 17.12 17.121 15z"></path><path fill="#141824" fill-rule="evenodd" d="M10.5 19a8.5 8.5 0 100-17 8.5 8.5 0 000 17zm0-2.75a5.75 5.75 0 100-11.5 5.75 5.75 0 000 11.5z" clip-rule="evenodd"></path></svg>'}</svelte:fragment
    >
  </Svelecte><br /> -->
<br><br>
  <!-- <AutofillSelector></AutofillSelector> -->
  <button class="toggle" class:notChosen={merchant} on:click={merchantClick}
    >Cardholders in {pca ?pca : "postal district"}</button
  >
  <button class="toggle" class:notChosen={cardholder} on:click={cardholderClick}
    >Merchants in {pca ? pca : "postal district"}</button
  >
  <br /><br />
  <div class="cont">
 

  <!--  <Switch
    bind:value={sliderValue}
    label="merchants in {pca}"
    label2="cardholders in {pca}"
    fontSize={18}
    design="slider"
  /> -->
    <p>
      {#if sliderValue == "on"}
        Where cardholders resident in {pca} spent their money in Q{timePeriod.split(
          "Q"
        )[1]}-{timePeriod.split("Q")[0]}
      {/if}
      {#if sliderValue == "off"}
        Merchants located in {pca}: where their customers came from in Q{timePeriod.split(
          "Q"
        )[1]}-{timePeriod.split("Q")[0]}
      {/if}
    </p>
  </div>
  <RangeSlider
    id="range"
    values={[periods.indexOf(timePeriod)]}
    min="0"
    max={periods.length - 1}
    pips
    float={true}
    pipstep="1"
    all="label"
    formatter={(v) => {
      let split = periods[v].split("Q");
      let quarter = "Q" + split[1];
      let year = split[1] == 1 ? split[0] : " ";
      return quarter + "<br>" + year;
    }}
    on:change={(e) => (timePeriod = periods[e.detail.value])}
    springValues={{ stiffness: 0.9, damping: 0.9 }}
    bind:value={timePeriod}
  />
  <br />
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

  <MyMap
    {timePeriod}
    {from}
    {to}
    {pca}
    {temp}
    bind:selected={pca}
    {decimals}
    {w}
  ></MyMap>


  <p>Source: Aggregated and anonymised data on UK card payments provided by Visa Europe Limited (2023)</p>
{/if}

<style>
  /* :global(#selector0--selection){
    background-color: blue;
  } */
  :global(#slider) {
    margin-bottom: 4.5em;
  }

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
  :global(body) {
    max-width: 800px;
  }

  :global(.pipVal) {
    transform: translate(0, 0) !important;
    font-size: 14px;
  }
  :global(.rangeFloat) {
    display: none !important;
  }
  .toggle {
    background-color: #206095;
    color: white;
    width: calc(50% - 0px);
    margin: none;
    border-radius: 2px;
    height: 45px;
    font-size: 16px;
    font-weight: 700;
    font-family: "Open Sans", sans-serif;
    float: left;
    border: 2px solid #206095;
    cursor: pointer;
  }
  .notChosen {
    background-color: white;
    font-weight: 400;
    color: #206095;
  }
  :global(#selector0) {
  width:100%;
    border: 2px solid #206095 !important;
    cursor: pointer;
    height: 40px;
  }
  :global(#selector0:hover) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(#selector0:focus) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(#selector0:active) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(.toggle:hover) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(.toggle:focus) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(.toggle:active) {
    box-shadow: 0 0 0 3pt #fbc900;
  }

  :global(.rangeNub) {
    background-color: #206095 !important;
  }
  :global(select){
background-size: 40px 40px !important;
  }
</style>
