<script>
  import Player from "./Player.svelte";
  import POI from "./POI.svelte";

  const mapWidth = 512;
  const mapHeight = 512;

  let playerSize = 32;
  let playerX = mapWidth / 2 - playerSize / 2;
  let playerY = mapHeight / 2 - playerSize / 2;

  let pois = [
    {
      name: "Computer",
      x: 32,
      y: mapHeight / 2 - 16,
      icon: "src/assets/pc.png",
      link: "",
    },
    {
      name: "Gatto",
      x: mapWidth - 64,
      y: mapHeight / 2 - 16,
      icon: "https://placekitten.com/32/32",
      link: "",
    },
  ];
</script>

<div class="fixed inset-0 w-screen h-screen bg-green-200 overflow-hidden">
  <Player bind:x={playerX} bind:y={playerY} />
  {#each pois as poi}
    <POI {...poi} {playerX} {playerY} />
  {/each}
</div>
