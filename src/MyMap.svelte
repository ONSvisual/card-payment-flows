<script>
	// Components for working with Mapbox layers
	import { getData, getColor, getTopo } from "./js/utils.js";
	import Map from './Map.svelte';
	import MapSource from './MapSource.svelte';
	import MapLayer from './MapLayer.svelte';
	import MapTooltip from './MapTooltip.svelte';
	export let timePeriod, direction, pca, from, to, selected, temp, w;

	$: console.log("timePeriod",timePeriod, "direction",direction, "pca", pca)

	const colors = {
		seq5: ['rgb(234, 236, 177)', 'rgb(169, 216, 145)', 'rgb(0, 167, 186)', 'rgb(0, 78, 166)', 'rgb(0, 13, 84)'],
		div10: ['#67001f','#b2182b','#d6604d','#f4a582','#fddbc7','#d1e5f0','#92c5de','#4393c3','#2166ac','#053061']	
	};

  let pcaData = (code) =>{	
	let folder=code.substr(0,2);
	let file=code.substr(0,4);
	//https://cdn.statically.io/gh/:user/:repo/:tag/:file
	return `https://cdn.statically.io/gh/ONSvisual/card-payment-flows/main/dist/data/${from}/${folder}/${file}.csv`//https://cdn.statically.io/gh/ONSvisual/card-payment-flows/dist/data/${from}/${folder}/${file}.csv` //`data2/${from}/${folder}/${file}.csv`;   //UPDATE THIS AT PUBLISHING!!!!!!!!!!!!!!!!!!!!!!
}

  const postalBounds = {
	  url: "./data2/postal.json",    //UPDATE THIS AT PUBLISHING!!!!!!!!!!!!!!!!!!!!!!
	  layer: "postal",
	  code: "PostDist"
	};

	


	const bounds = {
		uk: [[ -9, 46 ], [ 2, 59 ]],
		ew: [[-6, 49], [2, 56]]
	};

	const baseMaps = [
		{
			key: "omt",
			label: "OpenMapTiles",
			path: "./data2/style-ons-light.json"
		},
		{
			key: "osm",
			label: "OpenStreetMap",
			path: "./data2/style-osm-grey.json"
		},
	];
	
	// Bindings
	let map1, map2, map3, map4;

	// Data
	let data = {};
	let geojson;

	// State
	let zoom;
	let center = {};
	let hovered;

	let showSources = true;
	let showLayers = true;
	let visLayers = true;
	let baseMap = baseMaps[0];

	// Get geometry for geojson maps
	getTopo(postalBounds.url, postalBounds.layer)
	.then(res => geojson = res)
selected=pca

let breaks
	// Get data for geojson maps
 function renderMap(selected, from, to, timePeriod){
	getData(pcaData(selected))
	.then(res => {
		//let dataPCA=res.map(e=>{e[`${from}_location`]=Math.round(e[`${from}_location`]*1000)/1000; return e})
		let filterPca=res.filter(e=>e[`${from}_location`]==pca)
		let filterPcaTime=filterPca.filter(e=>e.time_period_value==timePeriod)
		let vals = filterPcaTime.map(d => d[`${from}_index_spend`]).sort((a, b) => a - b);
		let len = vals.length;
		breaks = [
			vals[0],
			vals[Math.floor(len * 0.2)],
			vals[Math.floor(len * 0.4)],
			vals[Math.floor(len * 0.6)],
			vals[Math.floor(len * 0.8)],
			vals[len - 1]
		];

		filterPcaTime.forEach(d => {
			d.color = getColor(d[`${from}_index_spend`], breaks, colors.seq5);
		});
		data.pca = filterPcaTime;
	});
 }

 $: renderMap(selected, from, to, timePeriod)
	// // Get data for vector tiles map
	// getData(lsoaData)
	// .then(res => {
		// values.forEach(d => {
		// 	d.color = colors.div10[+d.cardholder_index_spend - 1];
		// 	d.AREACD = d.cardholder_location;
	 	// });
	// 	data.lsoa = res;
	// });
$: console.log("breaks",breaks)
$: console.log("d",data )
</script>

{#if data && breaks}
			<div class="map">
			  {#if data && geojson 
			  }
			  <Map id="map3" style={baseMap.path} location={{bounds: bounds.uk}} bind:map={map3} controls={true}>
					{#if showSources}
			  	<MapSource
					  id="pca"
					  type="geojson"
					  data={geojson}
					  promoteId={postalBounds.code}
					  maxzoom={13}
					  
					  >
						{#if showLayers && temp}
					  <MapLayer
					  	id="pca-fill"
						idKey={`${to}_location`}
					  	data={data.pca}
					  	type="fill"
					  	hover={true}
					  	bind:hovered
					  	select={true}
					  	bind:selected
					  	paint={{
					  		'fill-color': ['case',
					  			['!=', ['feature-state', 'color'], null], ['feature-state', 'color'],//THIS IS AFFECTING THE PAINTING
					  			'rgba(255, 255, 255, 0)'
					  		],
					  		'fill-opacity': 0.7
				  		}}
							order={"place_town"}
							visible={visLayers}
				    >
						  <MapTooltip content={`${hovered}: 
						  ${data.pca.find(e=>e[to+"_location"]==hovered 
						  && e.time_period_value==timePeriod)?
						  Math.round(data.pca.find(e=>e[to+"_location"]==hovered 
						  && e.time_period_value==timePeriod)[from+"_index_spend"]*1000)/1000
						  :"unavailable"}`}/>
						</MapLayer>
				  	<MapLayer
				  		id="pca-line"
						idKey={`${to}_location`}
				  		type="line"
				  		paint={{
				  			'line-color': ['case',
				  				['==', ['feature-state', 'selected'], true], 'black',
				  				['==', ['feature-state', 'hovered'], true], 'orange',
				  				'rgba(255, 255, 255, 0)'
				  			],
				  			'line-width': ['case',
				  				['==', ['feature-state', 'selected'], true], 2,
				  				1
				  			]
				  		}}
							visible={visLayers}
						/>
						{/if}
				  </MapSource>
					{/if}
			  </Map>
			  {/if}
			</div>


<svg style="width:100%;">
{#each breaks as brk, i}
{#if i<5}
<rect x={i*(w/5)} y=10 width={w/5} height=50 fill={colors.seq5[i]}></rect>
{/if}
<line
x1={i*(w/5)}
y1=63
x2={i*(w/5)}
y2=10
stroke=black
>

</line>
<text 
x={i*(w/5)}
y=80 
fill
text-anchor={i==5?"end":i?"middle":"start"}
font-size=14
>{Math.round(brk*1000)/1000}</text>

{/each}
<text 
x={w}
y=99
fill
text-anchor=end
font-size=14
>Q1 2019 total = 100</text>
</svg>

{/if}
<style>
	section {
		display: -webkit-box;
		display: -ms-flexbox;
		display: flex;
		-webkit-box-pack: center;
		-ms-flex-pack: center;
		justify-content: center;
	  background-position: center;
	  background-repeat: no-repeat;
	  background-size: cover;
	  margin: 0;
		margin-bottom: 20px;
	  padding: 0;
	}
	button {
		padding: 0 2px;
		cursor: pointer;
	}
	label {
		display: inline;
		margin-left: 6px;
	}
	.wrapper {
		width: 100%;
		max-width: 768px;
		margin: 0 16;
	}
	.grid {
		display: grid;
		width: 100%;
		max-width: 768px;
		margin: 0 16;
		grid-gap: 30px;
		grid-template-columns: repeat(auto-fit, minmax(min(280px, 100%), 1fr));
		justify-content: stretch;
	}

	a:hover {
		cursor: pointer;
	}
	h2 {
		font-size: 1.2em;
		font-weight: bold;
	}
</style>
