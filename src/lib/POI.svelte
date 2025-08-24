<script>
  import { onMount, onDestroy } from "svelte";

  export let name;
  export let x;
  export let y;
  export let icon;
  export let link = "";
  export let playerX;
  export let playerY;
  export let poiSize;
  export let playerSize;

  let enterFrame = 0;
  let enterTimer;

  const enterFrames = 2;
  const enterFrameWratio = 0.47;
  const enterFrameHratio = 0.18;
  const enterAnimSpeed = 500;
  const imgSize = 0.053 * window.innerWidth;

  $: enterFrameW = poiSize * enterFrameWratio;
  $: enterFrameH = poiSize * enterFrameHratio;
  $: if (isNear) startEnterAnim();
  $: if (!isNear) stopEnterAnim();

  $: isNear =
    playerX < x + poiSize &&
    playerX + playerSize > x &&
    playerY < y + poiSize &&
    playerY + playerSize > y;

  function startEnterAnim() {
    if (enterTimer) return;
    enterTimer = setInterval(() => {
      enterFrame = (enterFrame + 1) % enterFrames;
    }, enterAnimSpeed);
  }
  function stopEnterAnim() {
    clearInterval(enterTimer);
    enterTimer = null;
    enterFrame = 0;
  }

  function interact() {
    if (link) window.open(link, "_blank");
    else alert(`Hai interagito con ${name}!`);
  }

  function handleKey(e) {
    if (isNear && e.key === "Enter") interact();
  }

  onMount(() => window.addEventListener("keydown", handleKey));
  onDestroy(() => window.removeEventListener("keydown", handleKey));
</script>

<div
  class="absolute flex flex-col items-center left-0 top-0"
  style="left:{x}px; top:{y}px; width:{poiSize}px; height:{poiSize}px; min-width:72px; min-height:72px;"
>
  <img
    src={icon}
    alt={name}
    class="transition-shadow duration-300 poi w-2/3 max-w-[120px] h-auto"
    style="width:{imgSize}px; max-width:100%;"
  />
  {#if isNear}
    <div
      class="absolute left-1/2 -translate-x-1/2 -top-8 sm:-top-10 md:-top-12"
      style="
        width: {enterFrameW}px;
        height: {enterFrameH}px;
        pointer-events: none;
        z-index: 30;
        background-image: url('src/assets/enter.png');
        background-repeat: no-repeat;
        background-position-x: -{enterFrame * enterFrameW}px;
        background-position-y: 0;
        background-size: {enterFrameW * enterFrames}px {enterFrameH}px;
        image-rendering: pixelated;
      "
    ></div>
  {/if}
</div>

<style>
  .poi {
    image-rendering: pixelated;
  }
</style>
