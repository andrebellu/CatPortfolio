<script>
    import { onMount, onDestroy } from "svelte";

    export let poi;
    export let playerX;
    export let playerY;
    export let poiSize;
    export let playerSize;

    let enterFrame = 0;

    let enterTimer;

    const enterFrames = 2;
    const enterFrameWratio = 0.8;
    const enterFrameHratio = 0.3;
    const enterAnimSpeed = 500;
    const imgSize = 0.053 * window.innerWidth;

    $: enterFrameW = poiSize * enterFrameWratio;

    $: enterFrameH = poiSize * enterFrameHratio;

    $: if (isNear) startEnterAnim();

    $: if (!isNear) stopEnterAnim();

    $: isNear =
        playerX < poi.x + poiSize &&
        playerX + playerSize > poi.x &&
        playerY < poi.y + poiSize &&
        playerY + playerSize > poi.y;

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
        if (poi.link) window.open(poi.link, "_blank");
        else alert(`Hai interagito con ${poi.name}!`);
    }

    function handleKey(e) {
        if (isNear && e.key === "Enter") interact();
    }

    onMount(() => window.addEventListener("keydown", handleKey));

    onDestroy(() => window.removeEventListener("keydown", handleKey));

    $: distance = Math.sqrt(
        Math.pow(poi.x - playerX, 2) + Math.pow(poi.y - playerY, 2)
    );

    $: isClose = distance < 145;
</script>

<div
    class="absolute transition-all duration-300"
    style="left: {poi.x - poiSize / 2}px; top: {poi.y -
        poiSize / 2}px; width: {poiSize}px; height: {poiSize}px;"
>
    <img
        src={poi.icon}
        alt={poi.name}
        class="w-full h-full object-contain"
        style="image-rendering: pixelated;"
    />

    {#if isClose}
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

{#if isClose && poi.link}
    <a
        href={poi.link}
        class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-blue-500 text-white px-4 py-2 rounded-full opacity-0 hover:opacity-100 transition-opacity duration-300"
        style="margin-top: {imgSize}px;"
    >
        Vai a {poi.name}
    </a>
{/if}
