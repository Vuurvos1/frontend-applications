<script>
  import HexMap from './components/hexmap.svelte';
  import Barchart from './components/barchart.svelte';

  import Info from './components/icons/info.svelte';

  let geoData;
  let points;

  let width;

  async function fetchData() {
    // check if there is saved data
    if (localStorage.getItem('geoData') && localStorage.getItem('points')) {
      geoData = JSON.parse(localStorage.getItem('geoData'));
      points = JSON.parse(localStorage.getItem('points'));
    } else {
      // fetch and save data if not found
      const geoDataUrl =
        'https://cartomap.github.io/nl/wgs84/arbeidsmarktregio_2020.geojson';
      const pointsUrl = `https://gist.githubusercontent.com/Vuurvos1/0f2e26a2e24732991f0d5b0120d8bb99/raw/24aa0e4fb59a3334b70aa8f037a601106201e773/geoSpecCombined.json`;

      [geoData, points] = await Promise.all([
        await (await fetch(geoDataUrl)).json(),
        await (await fetch(pointsUrl)).json(),
      ]);

      // save data to local storage
      localStorage.setItem('geoData', JSON.stringify(geoData));
      localStorage.setItem('points', JSON.stringify(points));
    }

    return [geoData, points];
  }
</script>

<style>
  :global(h1, h2, h3, h4) {
    font-family: 'Ubuntu', sans-serif;
    font-weight: 300;

    margin-bottom: 0.6em;
  }
  :global(h1) {
    font-size: 3rem;

    margin-bottom: 0.8em;
  }

  :global(h2) {
    font-size: 2rem;
  }

  :global(h3) {
    font-size: 1.5rem;
  }

  :global(p, a, text) {
    font-family: 'Roboto', sans-serif;
  }

  :global(p) {
    margin-bottom: 0.8em;
  }

  :global(a) {
    text-decoration: underline;
    color: var(--black);
  }

  :global() header {
    margin: 0 auto;
    padding: 3.6rem 1.2rem 0 1.2rem;

    max-width: 35rem;
  }

  main {
    display: flex;
    flex-direction: column;

    margin: 0 auto;
    padding: 0 1.2rem;

    max-width: 35rem;
  }

  main section {
    margin-bottom: 2rem;
  }

  .infoText {
    color: var(--white);
    background-color: var(--error);

    margin-bottom: 1.4em;
    padding: 0.5rem 0.8rem 0.8rem 0.8rem;

    border-radius: 0.5rem;
  }

  #bronnen {
    margin-bottom: 3rem;
  }

  #bronnen ul {
    list-style-position: inside;
  }

  #bronnen li {
    margin-bottom: 0.2em;
  }
</style>

<svelte:head>
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link
    href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500&family=Ubuntu:wght@300;400;500;700&display=swap"
    rel="stylesheet" />
</svelte:head>

<header>
  <h1>Hoe zit het met de parkeer dichtheid binnen Nederland?</h1>
</header>

<main>
  <section>
    <p>
      Er zijn meer dan 15 miljoen parkeerplaatsen binnen Nederland. Dat is iets
      meer dan een parkeerplek per Nederlanders, of 1,8 parkeerplaatsen per
      autos. Maar hoe zijn deze parkeerplaatsen dan verdeeld over Nederland?
    </p>

    <p>
      De meeste parkeerplekken binnen Nederland zijn echter geen parkeergarages
      maar plaatsen langs de weg.
    </p>

    {#await fetchData()}
      <h2>Loading Map</h2>
    {:then data}
      <p class="infoText">
        <Info />
        Data is niet genormaliseerd aan de hand van de bevolkingsdichtheid in
        een gebied.
      </p>

      <HexMap {data} />

      <p>
        Je ziet misschien dat een groot van de zeshoekjes geen kleur heeft wil
        niet zeggen dat er hier geen parkeerplaatsen zijn, de meeste
        parkeerplaatsen in Nederland bevinden zich namelijk langs de straat.
      </p>
    {:catch error}
      <p style="color: red">{error.message}</p>
    {/await}
  </section>

  <section bind:clientWidth={width}>
    <h2>Capaciteit per garage</h2>

    <Barchart {width} />
  </section>

  <section id="bronnen">
    <h3>Sources</h3>

    <ul>
      <li><a href="https://opendata.rdw.nl/">Datasets van Opendata RDW</a></li>
      <li>
        <a href="https://github.com/vuurvos1/frontend-applications">Source code
          op Github
        </a>
      </li>
    </ul>
  </section>
</main>
