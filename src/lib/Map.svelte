<script>
  import Player from "./Player.svelte";
  import POI from "./POI.svelte";
  import { POIS } from "./pois.js";
  import { onMount } from "svelte";

  let mapWidth = 0;
  let mapHeight = 0;
  let showDebug = false;
  let collisionDebug = [];

  const readSize = () => {
    mapWidth = window.innerWidth;
    mapHeight = window.innerHeight;
  };

  onMount(() => {
    readSize();
    window.addEventListener("resize", readSize);

    const handleKey = (e) => {
      if (e.key === "k" || e.key === "K") {
        showDebug = !showDebug;
      }
    };
    window.addEventListener("keydown", handleKey);

    return () => {
      window.removeEventListener("resize", readSize);
      window.removeEventListener("keydown", handleKey);
    };
  });

  // dimensioni dinamiche
  $: playerSize = 0.018 * mapWidth;
  $: poiSize = 0.1 * mapWidth;
  const factor = 0.53;

  // player al centro
  let playerX = 0;
  let playerY = 0;
  $: playerX = mapWidth / 2 - playerSize / 2;
  $: playerY = mapHeight / 2 - playerSize / 2;

  // pois
  $: pois = POIS.map((poi) => ({
    ...poi,
    x: poi.relX * mapWidth,
    y: poi.relY * mapHeight,
  }));

  function checkCollision(newX, newY) {
    const playerRect = { x: newX, y: newY, w: playerSize, h: playerSize };
    let hit = false;

    for (const poi of pois) {
      const hb = poi.hitbox ?? { w: 1, h: 1, offsetY: 0 };

      const actualW = poiSize * hb.w;
      const actualH = poiSize * hb.h;

      const poiRect = {
        x: poi.x + (poiSize - actualW) / 2,
        y: poi.y + (poiSize - actualH) / 2 + hb.offsetY * poiSize,
        w: actualW,
        h: actualH,
      };

      const overlaps =
        playerRect.x < poiRect.x + poiRect.w &&
        playerRect.x + playerRect.w > poiRect.x &&
        playerRect.y < poiRect.y + poiRect.h &&
        playerRect.y + playerRect.h > poiRect.y;

      collisionDebug = [
        ...collisionDebug,
        { poi: poi.name, overlaps, ts: Date.now() },
      ];

      if (overlaps) hit = true;
    }

    return hit;
  }
</script>

<div class="fixed inset-0 w-full h-full bg-green-200 overflow-hidden">
  <Player
    bind:x={playerX}
    bind:y={playerY}
    on:beforeMove={(e) => {
      const { newX, newY, cancel } = e.detail;
      if (checkCollision(newX, newY)) cancel();
    }}
    debug={showDebug}
  />

  {#each pois as poi}
    <POI
      name={poi.name}
      x={poi.x}
      y={poi.y}
      icon={poi.icon}
      link={poi.link}
      {playerX}
      {playerY}
      {poiSize}
      {playerSize}
    />
  {/each}

  {#if showDebug}
    <!-- Player hitbox -->
    <div
      class="absolute border-2 border-red-500 bg-red-200 opacity-50 pointer-events-none"
      style="left:{playerX}px; top:{playerY}px; width:{playerSize}px; height:{playerSize}px;"
    ></div>

    <!-- POI hitbox -->
    {#each pois as poi}
      <div
        class="absolute border-2 border-blue-500 bg-blue-200 opacity-40 pointer-events-none"
        style="
        left:{poi.x + (poiSize - poiSize * (poi.hitbox?.w ?? 1)) / 2}px;
        top:{poi.y +
          (poiSize - poiSize * (poi.hitbox?.h ?? 1)) / 2 +
          (poi.hitbox?.offsetY ?? 0) * poiSize}px;
        width:{poiSize * (poi.hitbox?.w ?? 1)}px;
        height:{poiSize * (poi.hitbox?.h ?? 1)}px;
      "
      >
        <div
          class="absolute -top-6 left-0 text-xs bg-white px-1 border border-gray-300"
        >
          {poi.name}
        </div>
      </div>
    {/each}

    <!-- Debug info -->
    <div
      class="absolute top-4 left-4 bg-black bg-opacity-80 text-white p-4 rounded text-xs max-w-md z-50"
    >
      <h3 class="font-bold mb-2">DEBUG INFO (Premi K per nascondere)</h3>
      <div>Player: {Math.round(playerX)}, {Math.round(playerY)}</div>
      <div>Player Size: {playerSize.toFixed(1)}px</div>
      <div>POI Size: {poiSize.toFixed(1)}px (factor {factor})</div>

      <h4 class="font-bold mt-2 mb-1">Ultime collisioni:</h4>
      {#each collisionDebug.slice(-5).reverse() as debug}
        <div
          class="mt-1 p-1 rounded {debug.overlaps
            ? 'bg-red-600'
            : 'bg-green-600'}"
        >
          {new Date(debug.ts).toLocaleTimeString()} → {debug.poi}:{" "}
          {debug.overlaps ? "COLLISIONE" : "OK"}
        </div>
      {/each}
    </div>
  {/if}
</div>
