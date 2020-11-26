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
      const pointsUrl = `https://gist.githubusercontent.com/Vuurvos1/0f2e26a2e24732991f0d5b0120d8bb99/raw/a1f2c961e1c0c6bb4f17185b7c11dca240d51e2a/geoSpecCombined.json`;

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
  }
  :global(h1) {
    font-weight: 300;
    font-size: 3rem;

    margin-bottom: 0.8em;
  }

  :global(h2) {
    font-weight: 300;
    font-size: 2.25rem;

    margin-bottom: 0.6em;
  }

  :global(p, a) {
    font-family: 'Open Sans', sans-serif;
  }

  :global(p) {
    margin-bottom: 0.8em;
  }

  :global(a) {
    text-decoration: underline;
    color: var(--black);
  }

  header {
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

  #bronnen {
    margin-bottom: 3rem;
  }
</style>

<header>
  <h1>Hoe zit het met de parkeer dichtheid binnen Nederland?</h1>
</header>

<main>
  <section>
    <p>
      Er zijn x aantal parkeerplaatsen binnen Nederland, Dat is 1 parkeer plek
      per x Nederlanders of 1 parkeerplaats per x aantal autos. Maar hoe zijn
      deze parkeerplaatsen dan verdeeld over Nederland?
    </p>

    <p>
      De meeste parkeerplekken binnen Nederland zijn echter geen parkeer garages
      maar plaatsen langs de weg
    </p>

    {#await fetchData()}
      <h2>Loading Map</h2>
    {:then data}
      <p>
        <Info />
        Data is niet genormalizeerd aan de hand van de bevolkingsdichtheid in
        een gebied.
      </p>

      <HexMap {data} />

      <p>
        Dat een van de zeshoekjes geen kleur heeft wil niet zeggen dat er hier
        geen parkeerplaatsen zijn, de meeste parkeerplaatsen in Nederland zijn
        langs de straat en deze vizualizatie kijkt alleen naar de parkeer
        garages binnen Nederland.
      </p>
    {:catch error}
      <p style="color: red">{error.message}</p>
    {/await}
  </section>

  <section bind:clientWidth={width}>
    <h2>Capaciteit per garage</h2>

    <Barchart {width} />
  </section>

  <!-- <section>
    <h2>Toegankelijkheid Garages</h2>
    {#await fetchData()}
      <h2>Loading Map</h2>
    {:then data}
      <HexMap {data} />
    {:catch error}
      <p style="color: red">{error.message}</p>
    {/await}

    <p>
      Uit deze data blijkt dat maar 13 van de 355 parkeer garages toegankelijk
      zijn voor rolstoelen
    </p>
  </section> -->

  <section id="bronnen">
    <h2>Sources</h2>

    <a href="https://opendata.rdw.nl/">Datasets van Opendata RDW</a>
    <a href="https://github.com/vuurvos1/frontend-applications">Bekijk de source
      code op Github
    </a>
  </section>
</main>
