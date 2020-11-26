<script>
  import { scaleLinear, max } from 'd3';

  import { barData } from './../store';

  export let width;

  let barcharData;
  const barPadding = 10;
  const height = 450;

  const barMargin = {
    left: 200,
    right: 0,
    top: 0,
    bottom: 35,
  };

  barData.subscribe((value) => {
    barcharData = [];

    for (let i = 0; i < value.datapoints; i++) {
      barcharData.push({
        value: Number(value[i].capacity),
        label: value[i].areadesc,
      });
    }
  });

  // setup barchart scales
  $: xScale = scaleLinear()
    .domain([0, max(barcharData, (d) => d.value)])
    .range([0, width - barMargin.left - barMargin.right]);

  $: maxCap = max(barcharData, (d) => d.value);
  $: xTicks = () => {
    let ticks = [];

    for (let i = 0; i < maxCap; i++) {
      if (maxCap < 150) {
        if (i % 10 == 0) {
          ticks.push(i);
        }
      } else if (maxCap < 500) {
        if (i % 50 == 0) {
          ticks.push(i);
        }
      } else {
        if (i % 100 == 0) {
          ticks.push(i);
        }
      }
    }

    return ticks;
  };

  $: barWidth =
    (height - barMargin.bottom - barcharData.length * barPadding) /
    barcharData.length;
  $: barOffset = (height - barMargin.bottom) / barcharData.length;

  function yValue(value) {
    // if string to long chop of text between brackets
    if (value.length < 25) {
      return value;
    } else {
      return value.replace(/(\(.*?\))/g, '').trim();
    }
  }
</script>

<style>
  .barchart rect {
    fill: var(--highlight);
  }

  .barchart text {
    font-family: 'Open Sans', sans-serif;
    font-weight: 400;
    font-size: 0.825rem;
  }

  .barchart .xAxis line {
    stroke: currentColor;
  }
</style>

{#if barcharData.length > 0}
  <svg class="barchart" {width} {height}>
    <g>
      {#each barcharData as bar, i}
        <rect
          x={barMargin.left}
          y={i * barOffset}
          width={xScale(bar.value)}
          height={barWidth} />
      {/each}
    </g>

    <g
      class="xAxis"
      transform={`translate(${barMargin.left}, ${height - barMargin.bottom})`}>
      <line x2={width - barMargin.left} />
      {#each xTicks() as tick, i}
        <g
          transform={`translate(${((width - barMargin.left - barMargin.right) / xTicks().length) * i}, 0)`}>
          <line y2="12" />
          <text y="28" text-anchor="middle">{tick}</text>
        </g>
      {/each}
    </g>

    <g class="yAxis">
      {#each barcharData as bar, i}
        <text
          text-anchor="end"
          x={barMargin.left - 10}
          y={barOffset * i + barWidth / 2}>
          {yValue(bar.label)}
        </text>
      {/each}
      <rect />
    </g>
  </svg>
{:else}
  <p>
    Klik op een van de geleurde zeshoekjes in de vizualizatie hierboven voor
    meer informatie
  </p>
{/if}
