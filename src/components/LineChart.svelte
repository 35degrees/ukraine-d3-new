<script>
  import * as d3 from 'd3'

  export let data;
  export let width = 928;
  export let height = 500;
  export let marginTop = 40;
  export let marginRight = 50;
  export let marginBottom = 30;
  export let marginLeft = 20;
  export let category;
  export let yLabel;

  const colors = {
    "2022": "#3691c8",
    "2023": "#09938b",
    "2024": "#ce5a0e"
  }

  let previousYearTotal = 0

  $: processedData = data.map(obj => {
    return Object.fromEntries(
      Object.entries(obj).map(([key,value]) => [key,value === null ? 0 : value])
    )
  })

  $: dataByYears = d3.groups(processedData, d => d.date.getFullYear());


  $: summaryData = dataByYears.map(el => {
    const yearTotal = d3.max(el[1], d => d[category]);

    const adjustedYearTotal = yearTotal - previousYearTotal;


    const newData = {
      year: el[0],
      data: el[1].map(e => {
        return {
          ...e,
          extractedValue: el[0] == 2022 ? e[category] : e[category] - previousYearTotal
        }
      }),
      yearTotal: el[0] == 2022 ? yearTotal : adjustedYearTotal
    }
    previousYearTotal = yearTotal
    return newData
  })

$: xScale = d3.scaleLinear([1,365],[marginLeft,width - marginRight])
$: yScale = d3.scaleLinear([0,d3.max(summaryData, d => d.yearTotal)],[height-marginBottom, marginTop])

$:line = d3.line()
  .x(d => xScale(d.dayNumber))
  .y(d => yScale(d.extractedValue))

$:summaryData.forEach(v => {
  v.transform = `translate(${xScale(d3.max(v.data,d => d.dayNumber)) + 12},
  ${yScale(d3.max(v.data, d => d.extractedValue)) + 1})`
})

</script>

<svg
  {width}
  {height}
  viewBox="0 0 {width} {height}"
  style:max-width="100%"
  style:height="auto"
>

<g transform="translate(0,{height-marginBottom})">
  <line stroke="currentColor" x1={marginLeft} x2={width}/>
  {#each xScale.ticks() as tick}
    <line stroke="currentColor" x1={xScale(tick)} x2={xScale(tick)} y1={0} y2={6}/>
    <text fill="currentColor" text-anchor="middle" x={xScale(tick)} y={22}>
      {tick}
    </text>
  {/each}

  <text fill="currentColor" text-anchor="end" x={xScale(365) + marginRight} y={22}>
    Day
  </text>
</g>

<g transform="translate({marginLeft},0)">
  {#each yScale.ticks() as tick}
    {#if tick !== 0}
    <line stroke="currentColor" stroke-opacity="0.1" x1={0} x2={width-marginLeft} y1={yScale(tick)} y2={yScale(tick)}/>
    {/if}
    <text fill="currentColor" text-anchor="start" font-size="14px" dominant-baseline="middle" x={0} y={yScale(tick) - 10}>
      {tick}
    </text>
  {/each}
  {#if yLabel}
    <text fill="currentColor" text-anchor="start" x={0} y={15}>
      {yLabel}
    </text>
  {/if}
</g>

{#each summaryData as v}
  <path fill="none" stroke={colors[v.year]} stroke-width="2" style="transition: d 0.5s ease;" d={line(v.data)} />
  <circle 
    cx={xScale(d3.max(v.data,d=>d.dayNumber)) + 3}
    cy={yScale(d3.max(v.data,d=>d.extractedValue)) -3}
    r="5"
    stroke={colors[v.year]}
    stroke-width="3"
    fill="none"
    style="transition: cx 0.5s ease, cy 0.5s ease"
  />
  <text
  font-size="14px"
  fill={colors[v.year]}
  style="transition: transform 0.5s ease"
  >
    {v.year}
  </text>
{/each}
</svg>