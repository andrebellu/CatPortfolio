<script>
  export let name;
  export let x;
  export let y;
  export let icon;
  export let link = "";
  export let playerX;
  export let playerY;

  const playerSize = 32;
  const poiSize = 96;
  const imgSize = 96;

  $: isNear =
    playerX < x + poiSize &&
    playerX + playerSize > x &&
    playerY < y + poiSize &&
    playerY + playerSize > y;

  function interact() {
    if (link) window.open(link, "_blank");
    else alert(`Hai interagito con ${name}!`);
  }

  function handleKey(e) {
    if (isNear && e.key === "Enter") interact();
  }

  import { onMount, onDestroy } from "svelte";
  onMount(() => window.addEventListener("keydown", handleKey));
  onDestroy(() => window.removeEventListener("keydown", handleKey));
</script>

<div
  class="absolute flex flex-col items-center left-0 top-0"
  style="left:{x}px; top:{y}px; width:{poiSize}px; height:{poiSize}px;"
>
  <img
    src={icon}
    alt={name}
    class="transition-shadow duration-300 poi"
    class:shadow-[0_0_8px_4px_rgba(255,255,0,0.7)]={isNear}
    style="width:{imgSize}px; height:{imgSize}px;"
  />
  {#if isNear}
    <div
      class="text-black text-xs px-2 py-1 rounded mt-1 shadow z-20 bg-white/80"
    >
      Premi ENTER per interagire con {name}
    </div>
  {/if}
</div>

<style>
  .poi {
    image-rendering: pixelated;
  }
</style>
