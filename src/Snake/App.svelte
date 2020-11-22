<script>
  import { onMount } from 'svelte';

  let gridWidth = 15;
  let gridHeight = 10;

  let direction = [0, 1];

  let bigoof = false;

  let grid = [...Array(gridHeight)].map(() =>
    [...Array(gridWidth)].map(() => 'empty')
  );

  let snakePos = [[6, 5]];

  $: {
    snakePos.forEach(([x, y]) => {
      grid[x][y] = 'snake';
    });
  }

  grid[5][12] = 'food';

  grid[5][6] = 'snake';

  onMount(() => {
    setInterval(() => {
      snakePos[0][0] += direction[0];
      snakePos[0][1] += direction[1];

      if (
        snakePos[0][0] < 0 ||
        snakePos[0][1] < 0 ||
        snakePos[0][0] > gridHeight ||
        snakePos[0][1] > gridWidth
      ) {
        bigoof = true;
        console.log(bigoof);
      }

      console.log(snakePos[0]);
    }, 250);
  });
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  .row {
    display: flex;
    flex-direction: row;
  }

  .cell {
    width: 1rem;
    height: 1rem;

    background-color: hotpink;
    border: 1px solid white;
  }

  .food {
    background-color: green;
  }

  .snake {
    background-color: orangered;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<svelte:window
  on:keydown={(e) => {
    switch (e.key) {
      case 'ArrowLeft':
        direction = [0, -1];
        break;
      case 'ArrowRight':
        direction = [0, 1];
        break;
      case 'ArrowUp':
        direction = [-1, 0];
        break;
      case 'ArrowDown':
        direction = [1, 0];
        break;
    }
  }} />

<main>
  <p>
    Visit the
    <a href="https://svelte.dev/tutorial">Svelte tutorial</a>
    to learn how to build Svelte apps.
  </p>

  <div>
    {#each grid as row}
      <div class="row">
        {#each row as cell}
          <div class={`cell ${cell}`} />
        {/each}
      </div>
    {/each}
  </div>
</main>
