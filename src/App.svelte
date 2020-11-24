<script>
  import { onMount } from 'svelte';

  import HexMap from './components/hexmap.svelte';

  import {
    max,
    geoPath,
    geoMercator,
    scaleSequential,
    interpolateViridis,
    scaleLinear,
    scaleBand,
    axisLeft,
    axisBottom,
  } from 'd3';

  import { hexgrid } from 'd3-hexgrid';

  import { bindHexData, chartSetup } from './modules/chartHelpers';

  export let geoData;
  export let points;

  async function fetchData() {
    // check if there is saved data
    if (localStorage.getItem('geoData') && localStorage.getItem('points')) {
      geoData = JSON.parse(localStorage.getItem('geoData'));
      points = JSON.parse(localStorage.getItem('points'));
    } else {
      // fetch and save data if not found
      const geoDataUrl =
        'https://cartomap.github.io/nl/wgs84/arbeidsmarktregio_2020.geojson';
      const pointsUrl = `http://localhost:3000/garageGeo`;

      [geoData, points] = await Promise.all([
        await (await fetch(geoDataUrl)).json(),
        await (
          await fetch(pointsUrl, {
            method: 'POST',
          })
        ).json(),
      ]);

      // save data to local storage
      localStorage.setItem('geoData', JSON.stringify(geoData));
      localStorage.setItem('points', JSON.stringify(points));
    }

    return [geoData, points];
  }

  async function mainCode() {
    const url = `${window.location.origin}`;

    const geoDataUrl =
      'https://cartomap.github.io/nl/wgs84/arbeidsmarktregio_2020.geojson';
    // const pointsUrl = `${url}/garageGeo`;
    const pointsUrl = `http://localhost:3000/garageGeo`;

    // fetch data
    const [geoData, points] = await Promise.all([
      await (await fetch(geoDataUrl)).json(),
      await (
        await fetch(pointsUrl, {
          method: 'POST',
        })
      ).json(),
    ]);

    // do d3 stuff
    const mapSvg = chartSetup('#map svg', {
      top: 30,
      right: 30,
      bottom: 30,
      left: 30,
    });

    // Projection and path
    const projection = geoMercator().fitSize([mapSvg.w, mapSvg.h], geoData);
    const geoPath1 = geoPath().projection(projection);

    // Prep user data
    points.forEach((site) => {
      const coords = projection([+site.lng, +site.lat]);
      site.x = coords[0];
      site.y = coords[1];
    });

    // Create a hexgrid generator
    const hexgrid1 = hexgrid()
      .extent([mapSvg.w, mapSvg.h])
      .geography(geoData)
      .pathGenerator(geoPath1)
      .projection(projection)
      .hexRadius(5.2);

    // Instantiate the generator
    const hex = hexgrid1(points);

    // bind dataset to hexgrid hexagons
    bindHexData(hex, points);

    // Create exponential colorScale
    const colourScale = scaleSequential(function (t) {
      const tNew = Math.pow(t, 10);
      return interpolateViridis(tNew);
    }).domain([...hex.grid.extentPointDensity].reverse());

    // Draw the hexes
    mapSvg
      .append('g')
      .selectAll('path')
      .data(hex.grid.layout)
      .enter()
      .append('path')
      .attr('d', hex.hexagon())
      .attr('transform', (d) => `translate(${d.x} ${d.y})`)
      .style('fill', (d) =>
        !d.pointDensity ? '#fff' : colourScale(d.pointDensity)
      )
      .attr('class', (d) => {
        if (d.length > 0) {
          return 'point';
        }
      })
      .style('stroke', '#F4EB9F')
      .on('click', (e, d) => {
        d.datapoints > 0 ? drawBarChart(d) : null;
      })
      .append('title')
      .text((d) => {
        let totalCapacity = 0;
        for (let i = 0; i < d.length; i++) {
          totalCapacity += Number(d[i].capacity);
        }
        return `Total capacity: ${totalCapacity}`;
      });

    // initialize barchart
    const barMargin = {
      left: 240,
      right: 30,
      top: 20,
      bottom: 60,
    };

    const barSvg = chartSetup('#bar svg', barMargin);

    // setup barchart scales
    const xScale = scaleLinear().range([0, barSvg.w]);

    const yScale = scaleBand().range([0, barSvg.h]).padding(0.1);

    const g = barSvg
      .append('g')
      .attr('transform', `translate(${barMargin.left}, ${barMargin.top})`);

    barSvg
      .append('text')
      .attr('text-anchor', 'end')
      .attr('x', barSvg.w + barMargin.left - barMargin.right)
      .attr('y', barSvg.h + barMargin.top + 40)
      .text('Garage capacity');

    // setup axises
    const barY = g.append('g').call(axisLeft(yScale));
    const barX = g
      .append('g')
      .call(axisBottom(xScale))
      .attr('transform', `translate(0, ${barSvg.h})`);

    /**
     * Update an existing barchart with new data
     * @param {object} data - object array containg data from the hexagon
     */
    function drawBarChart(data) {
      const xValue = (d) => {
        return Number(d.capacity);
      };

      const yValue = (d) => {
        // if string to long chop of text between brackets
        if (d.areadesc.length < 25) {
          return d.areadesc;
        } else {
          return d.areadesc.replace(/(\(.*?\))/g, '').trim();
        }
      };

      const xScale = scaleLinear()
        .domain([0, max(data, xValue)])
        .range([0, barSvg.w]);

      const yScale = scaleBand()
        .domain(data.map(yValue))
        .range([0, barSvg.h])
        .padding(0.1);

      barY.call(axisLeft(yScale));
      barX.call(axisBottom(xScale));

      const u = g.selectAll('rect').data(data);

      u.enter()
        .append('rect')
        .merge(u)
        .transition()
        .attr('y', (d) => {
          return yScale(yValue(d));
        })
        .attr('width', (d) => {
          return xScale(xValue(d));
        })
        .attr('height', yScale.bandwidth());

      u.exit().remove();
    }
  }

  mainCode();
</script>

<style>
  :root {
    --blackL: #000;
    --white: #fff;
    --offWhite: #f8f8f8;

    --highlight: #6200ee;

    --dropShadow: 0px 16px 24px rgba(0, 0, 0, 0.14),
      0px 6px 30px rgba(0, 0, 0, 0.12), 0px 8px 10px rgba(0, 0, 0, 0.2);
  }

  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  body {
    min-height: 100vh;
    background-color: var(--offWhite);
  }

  h1 {
    font-family: 'Ubuntu', sans-serif;
    font-weight: 300;
    font-size: 3rem;

    margin-bottom: 0.4em;
    text-align: center;
  }

  h2 {
    font-family: 'Ubuntu', sans-serif;
    font-weight: 300;
    font-size: 2.25rem;

    margin-bottom: 0.4em;
  }

  p {
    font-family: 'Open Sans', sans-serif;
  }

  main {
    display: flex;
    flex-direction: column;
    /* display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 80vh;
    height: 100vh;
    width: 100vw; */
  }

  #bar {
    width: 100%;
    height: 100%;
    max-height: 32rem;
    margin: 1rem;

    position: relative;
  }

  #bar .info {
    position: absolute;
    width: 100%;
    height: 100%;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: var(--offWhite);
  }

  #bar .info p {
    text-align: center;
  }

  #bar svg {
    width: 100%;
    height: 100%;
  }

  .bar {
    box-shadow: var(--dropShadow);

    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    padding: 1rem 0;
  }

  .bar rect {
    fill: var(--highlight);
  }

  .bar text {
    font-family: 'Open Sans', sans-serif;
    font-weight: 400;
    font-size: 0.825rem;
  }

  /* Tablet */
  @media screen and (min-width: 48rem) {
    main {
      grid-template-columns: 1fr 1fr;
      height: 100vh;
      width: 100vw;
    }

    .map,
    .bar {
      height: 100vh;
    }
  }

  /* Desktop */
  @media screen and (min-width: 64rem) {
  }
</style>

<header>
  <h1>Parkeer dichtheid binnen Nederland</h1>
</header>

<main>
  <!-- <section class="map">
    <h1>Parking Density</h1>
    <div id="map"><svg /></div>
  </section>

  <section class="bar">
    <h2>Per garage capacity</h2>

    <div id="bar"> -->
  <!-- <div class="info">
        <p>Click on a coloured hexagon to see more details</p>
      </div> -->
  <!-- <svg />
    </div>
  </section> -->

  {#await fetchData()}
    <h2>Loading Map</h2>
  {:then data}
    <HexMap {data} />
  {:catch error}
    <p style="color: red">{error.message}</p>
  {/await}
</main>

<footer>
  <p>
    &copy; Sam de Kanter |
    <a href="https://github.com/vuurvos1/frontend-applications">Github Repo</a>
  </p>
</footer>
