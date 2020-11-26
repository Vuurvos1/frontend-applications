<script>
  import {
    geoPath,
    geoMercator,
    scaleSequential,
    interpolateViridis,
  } from 'd3';

  import { hexgrid } from 'd3-hexgrid';

  import { bindHexData } from './../modules/chartHelpers';

  export let data;

  const geoData = data[0];
  const points = data[1];

  const height = 620;
  const width = 500;

  // Projection and path
  const projection = geoMercator().fitSize([width, height], geoData);
  const geoPath1 = geoPath().projection(projection);

  // Prep user data
  points.forEach((site) => {
    const coords = projection([+site.lng, +site.lat]);
    site.x = coords[0];
    site.y = coords[1];
  });

  // Create a hexgrid generator
  const hexgrid1 = hexgrid()
    .extent([width, height])
    .geography(geoData)
    .pathGenerator(geoPath1)
    .projection(projection)
    .hexRadius(5.4);

  // Instantiate the generator
  const hex = hexgrid1(points);

  // bind dataset to hexgrid hexagons
  bindHexData(hex, points);

  // Create exponential colorScale
  const colourScale = scaleSequential((t) => {
    const tNew = Math.pow(t, 10);
    return interpolateViridis(tNew);
  }).domain([...hex.grid.extentPointDensity].reverse());

  function calculateCapacity(array) {
    let totalCapacity = 0;
    for (let i = 0; i < array.length; i++) {
      totalCapacity += Number(array[i].capacity);
    }
    return `Total capacity: ${totalCapacity}`;
  }

  // create array with color ticks
  const colors = [];
  for (let i = 0; i < 10; i++) {
    colors.push({ offset: `${i * 10}%`, col: `${colourScale(i / 100)}` });
  }

  import { barData } from './../store';

  function updateBarData(hex) {
    barData.set(hex);
  }
</script>

<style>
  .hexLegend {
    max-width: 10rem;
    height: auto;
  }

  .hexLegend rect {
    height: 0.6rem;
    width: 100%;

    fill: url(#hexGradient);
  }

  .point {
    cursor: pointer;
  }
</style>

<!-- map -->
<svg class="hexMap" viewBox={`0 0 ${width} ${height}`}>
  <g>
    {#each hex.grid.layout as hexagon}
      <path
        on:click={hexagon.datapoints > 0 ? () => updateBarData(hexagon) : {}}
        d={hex.hexagon()}
        transform={`translate(${hexagon.x} ${hexagon.y})`}
        fill={!hexagon.pointDensity ? '#fff' : colourScale(hexagon.pointDensity)}
        class={hexagon.length > 0 ? 'point' : ''}
        stroke="#F4EB9F">
        <title>{calculateCapacity(hexagon)}</title>
      </path>
    {/each}
  </g>
</svg>

<!-- legend -->
<svg height="16" class="hexLegend">
  <rect />

  <defs>
    <linearGradient id="hexGradient">
      {#each colors as col}
        <stop offset={col.offset} stop-color={col.col} />
      {/each}
    </linearGradient>
  </defs>
</svg>
