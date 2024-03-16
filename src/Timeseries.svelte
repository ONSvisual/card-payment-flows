<script>

    export let w, data, selected, date, options, height=70, series, transform
    
    let dates = series.map(e=>e.date)
    $: max=Math.max(...series.map(e=>e.amt))
    
    </script>
    <svg {w} {height} {transform} class="timeseries">
    {#each dates as dt, i}
    {#if dt==date}
    <line x1={(i/(dates.length-1))*w} x2={(i/(dates.length-1))*w} y1=0 y2={height-20} stroke="#206095" stroke-w="2px" opacity={0.3}></line>
    <!-- <text x={((i/(dates.length-1))*w)+5}  y=25 fill>£{series[i].amt?(series[i].amt/1000000).toLocaleString() +"M":0}</text> -->
    <circle cx={(i/(dates.length-1))*w} cy={series[i].amt?((((max-series[i].amt)/max)*(height-40) )+20) : (height-40)+20} r="5px" fill="black"/>
    {/if}
    {#if series[i+1]}
    <line x1={(i/(dates.length-1))*w} x2={((i+1)/(dates.length-1))*w} y1={series[i].amt?((((max-series[i].amt)/max)*(height-40) )+20) : (height-40)+20} y2={series[i+1].amt?((((max-series[i+1].amt)/max)*(height-40))+20) : (height-40)+20} stroke={series[i].amt&&series[i+1].amt?"#871A5B":"white"} stroke-width="3px"></line>
    {/if}
    {/each}
    <line x1=0 x2={w} y1={height-20} y2={height-20} stroke="grey"/>
    <text font-size=14 x=0 y={height-7} fill>{new Date(dates[0]*1000).toLocaleString("en-GB",options)}</text>
    <text font-size=14 x={w} y={height-7} text-anchor="end">{new Date(dates[dates.length-1]*1000).toLocaleString("en-GB",options)}</text>
    </svg>
    <style>
        .timeseries{
            background-color: white;
            overflow: visible;
            font-size: 14px;
        }
    </style>