<script>
    export let playerX;
    export let playerY;
    export let playerSize;
    export let poiSize;
    export let tileSize;
    export let collisionDebug = [];

    const fmt = (num) => Math.round(num);
</script>

<div
    class="absolute top-4 left-4 bg-black bg-opacity-80 text-white p-4 rounded text-lg max-w-md z-50"
>
    <h3 class="font-bold mb-2">DEBUG INFO (Premi K per nascondere)</h3>
    <div>Player: {fmt(playerX)}, {fmt(playerY)}</div>
    <div>Player Size: {fmt(playerSize)}px</div>
    <div>POI Size: {fmt(poiSize)}px</div>
    <div>Tile Size: {fmt(tileSize)}px</div>

    <h4 class="font-bold mt-2 mb-1">Ultime collisioni:</h4>
    {#each collisionDebug.slice().reverse() as debug, i}
        {#if i < 5}
            <div
                class="mt-1 p-1 rounded {debug.overlaps
                    ? 'bg-red-600'
                    : 'bg-green-600'}"
            >
                {new Date(debug.ts).toLocaleTimeString()} → {debug.poi}: {debug.overlaps
                    ? "COLLISIONE"
                    : "OK"}
            </div>
        {/if}
    {/each}
</div>
