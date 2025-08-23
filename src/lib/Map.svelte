<script>
  import { onMount } from "svelte";

  import Player from "./Player.svelte";
  import POI from "./POI.svelte";

  let mapWidth = window.innerWidth;
  let mapHeight = window.innerHeight;

  function updateMapSize() {
    mapWidth = window.innerWidth;
    mapHeight = window.innerHeight;
  }

  onMount(() => {
    window.addEventListener("resize", updateMapSize);
  });

  let playerSize = 0.018 * mapWidth;
  $: playerSize = 0.018 * mapWidth;
  let playerX = mapWidth / 2 - playerSize / 2;
  let playerY = mapHeight / 2 - playerSize / 2;

  $: pois = [
    {
      name: "Computer",
      x: 0.06 * mapWidth,
      y: mapHeight / 2 - 0.05 * mapWidth,
      icon: "src/assets/pc.png",
      link: "",
    },
    {
      name: "Gatto",
      x: mapWidth - 0.13 * mapWidth,
      y: mapHeight / 2 - 0.05 * mapWidth,
      icon: "https://placekitten.com/32/32",
      link: "",
    },
  ];
</script>

<div
  class="fixed inset-0 w-full h-full min-h-screen min-w-screen bg-green-200 overflow-hidden"
>
  <Player bind:x={playerX} bind:y={playerY} />
  {#each pois as poi}
    <POI {...poi} {playerX} {playerY} />
  {/each}
</div>
