<script>
  import { scaleLinear, scaleBand, max, range } from 'd3';

  import { barData } from './../store';

  let barcharData;

  export let width;

  barData.subscribe((value) => {
    barcharData = [];

    for (let i = 0; i < value.datapoints; i++) {
      barcharData.push({
        value: Number(value[i].capacity),
        label: value[i].areadesc,
      });
    }
  });

  //   const width = 500;
  const height = 450;

  const barMargin = {
    left: 120,
    right: 30,
    top: 20,
    bottom: 60,
  };

  // setup barchart scales

  $: xScale = scaleLinear()
    .domain([
      0,
      max(barcharData, (d) => {
        return d.value;
      }),
    ])
    .range([0, width]);

  $: yScale = scaleBand().range([0, height]).padding(0.1);

  //   $: yTicks = () => {
  //     const [startPoint, highestPoint] = yScale.domain();
  //     const ticks = [];
  //     for (let i = startPoint; i <= highestPoint; i++) {
  //       ticks.push(i);
  //     }
  //     return ticks;
  //   };

  $: innerWidth = width - (barMargin.left + barMargin.right);
  $: barWidth = innerWidth / barcharData.length;

  //   const g = barSvg
  //     .append('g')
  //     .attr('transform', `translate(${barMargin.left}, ${barMargin.top})`);

  //   barSvg
  //     .append('text')
  //     .attr('text-anchor', 'end')
  //     .attr('x', barSvg.w + barMargin.left - barMargin.right)
  //     .attr('y', barSvg.h + barMargin.top + 40)
  //     .text('Garage capacity');

  //   // setup axises
  //   const barY = g.append('g').call(axisLeft(yScale));
  //   const barX = g
  //     .append('g')
  //     .call(axisBottom(xScale))
  //     .attr('transform', `translate(0, ${barSvg.h})`);

  //  function drawBarChart(data) {
  //   const xValue = (d) => {
  //     return Number(d.capacity);
  //   };

  //   const yValue = (d) => {
  //     // if string to long chop of text between brackets
  //     if (d.areadesc.length < 25) {
  //       return d.areadesc;
  //     } else {
  //       return d.areadesc.replace(/(\(.*?\))/g, '').trim();
  //     }
  //   };

  //   const xScale = scaleLinear()
  //     .domain([0, max(data, xValue)])
  //     .range([0, barSvg.w]);

  //   const yScale = scaleBand()
  //     .domain(data.map(yValue))
  //     .range([0, barSvg.h])
  //     .padding(0.1);

  //   barY.call(axisLeft(yScale));
  //   barX.call(axisBottom(xScale));

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
</style>

<p>Klik op een van de geleurde zeshoekjes voor meer informatie</p>

<!-- <svg viewBox={`0 0 ${width} ${height}`}> -->
<svg class="barchart" {width} {height}>
  <g>
    {#each barcharData as bar, i}
      <rect
        class={bar.value}
        x={barMargin.left}
        y={i * (height / barcharData.length) + i * 10}
        width={xScale(bar.value)}
        height={height / barcharData.length} />
    {/each}
  </g>

  <g class="xAxis">
    <rect />
  </g>

  <g class="yAxis">
    {#each barcharData as bar, i}
      <text
        text-anchor="end"
        x={barMargin.left - 10}
        y={i * (height / barcharData.length) + height / barcharData.length / 2}>
        {yValue(bar.label)}
      </text>
    {/each}
    <rect />
  </g>
</svg>
