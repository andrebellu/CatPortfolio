<script>
    import Player from "./Player.svelte";
    import POI from "./POI.svelte";
    import Debug from "./Debug.svelte";
    import { POIS } from "./pois.js";
    import { onMount, tick } from "svelte";

    export let showDebug = false;

    let mapWidth = 0;
    let mapHeight = 0;
    let collisionDebug = [];

    $: tileSize = Math.min(mapWidth, mapHeight) / 15;
    $: playerSize = tileSize * 1.2;
    $: poiSize = tileSize * 1.7;

    let playerX = 0;
    let playerY = 0;

    $: playerHitbox = (() => {
        const scale = 0.7;
        const w = playerSize * scale;
        const h = playerSize * scale;
        const offsetX = -playerSize * 0.28;
        const offsetY = -playerSize * 0.18;
        return {
            x: playerX - w / 2 + offsetX,
            y: playerY - h / 2 + offsetY,
            w,
            h,
        };
    })();

    function recenterPlayer() {
        playerX = mapWidth / 2;
        playerY = mapHeight / 2;
    }

    const readSize = async () => {
        mapWidth = window.innerWidth;
        mapHeight = window.innerHeight;
        await tick();
        recenterPlayer();
    };

    onMount(() => {
        readSize();
        window.addEventListener("resize", readSize);

        const handleKey = (e) => {
            if (e.key === "k" || e.key === "K") showDebug = !showDebug;
        };
        window.addEventListener("keydown", handleKey);

        return () => {
            window.removeEventListener("resize", readSize);
            window.removeEventListener("keydown", handleKey);
        };
    });

    $: pois = POIS.map((poi) => {
        const hasRel =
            typeof poi.relX === "number" && typeof poi.relY === "number";
        const px = hasRel ? poi.relX * mapWidth : (poi.x ?? 0) * tileSize;
        const py = hasRel ? poi.relY * mapHeight : (poi.y ?? 0) * tileSize;
        return { ...poi, x: px, y: py };
    });

    function getPoiRect(poi) {
        const hb = poi.hitbox ?? { w: 1, h: 1, offsetX: 0, offsetY: 0 };
        const actualW = poiSize * (hb.w ?? 1);
        const actualH = poiSize * (hb.h ?? 1);
        const offsetX = (hb.offsetX ?? 0) * poiSize;
        const offsetY = (hb.offsetY ?? 0) * poiSize;

        return {
            x: poi.x - actualW / 2 + offsetX,
            y: poi.y - actualH / 2 + offsetY,
            w: actualW,
            h: actualH,
        };
    }

    function rectsOverlap(a, b) {
        return (
            a.x < b.x + b.w &&
            a.x + a.w > b.x &&
            a.y < b.y + b.h &&
            a.y + a.h > b.y
        );
    }

    function checkCollision(newX, newY) {
        const scale = 0.7;
        const w = playerSize * scale;
        const h = playerSize * scale;
        const offsetX = -playerSize * 0.28;
        const offsetY = -playerSize * 0.2;
        const testRect = {
            x: newX - w / 2 + offsetX,
            y: newY - h / 2 + offsetY,
            w,
            h,
        };

        let hitPoi = null;
        for (const poi of pois) {
            const pr = getPoiRect(poi);
            if (rectsOverlap(testRect, pr)) {
                hitPoi = poi;
                break;
            }
        }

        collisionDebug.push({
            poi: hitPoi ? hitPoi.name : "—",
            overlaps: !!hitPoi,
            ts: Date.now(),
        });
        if (collisionDebug.length > 10) collisionDebug.shift();

        return !!hitPoi;
    }

    function handleBeforeMove(e) {
        const { newX, newY, cancel } = e.detail;
        if (checkCollision(newX, newY)) {
            cancel();
        } else {
            playerX = newX;
            playerY = newY;
        }
    }
</script>

<div
    class="relative w-full h-full overflow-hidden border-2 border-black bg-center bg-cover"
    style="background-image: url('src/assets/mappa.png')"
>
    <Player
        x={playerX}
        y={playerY}
        size={playerSize}
        on:beforeMove={handleBeforeMove}
        debug={showDebug}
    />

    {#each pois as poi}
        <POI {poi} {playerX} {playerY} {poiSize} {playerSize} />
    {/each}

    {#if showDebug}
        <div
            class="absolute border-2 border-red-500 bg-red-200 opacity-50 pointer-events-none"
            style="left:{playerHitbox.x}px; top:{playerHitbox.y}px; width:{playerHitbox.w}px; height:{playerHitbox.h}px;"
        ></div>

        <div
            class="absolute pointer-events-none bg-yellow-400 z-50 rounded-sm"
            style="left:{playerX - 4}px; top:{playerY -
                4}px; width:8px; height:8px;"
        ></div>

        {#each pois as poi}
            {@const pr = getPoiRect(poi)}
            <div
                class="absolute border-2 border-blue-500 bg-blue-200 opacity-40 pointer-events-none"
                style="left:{pr.x}px; top:{pr.y}px; width:{pr.w}px; height:{pr.h}px;"
            >
                <div
                    class="absolute -top-6 left-0 text-xs bg-white px-1 border border-gray-300"
                >
                    {poi.name}
                </div>
            </div>

            <div
                class="absolute pointer-events-none bg-purple-600 z-40 rounded-sm"
                style="left:{poi.x - 4}px; top:{poi.y -
                    4}px; width:8px; height:8px;"
            ></div>
        {/each}

        <Debug
            {playerX}
            {playerY}
            {playerSize}
            {poiSize}
            {tileSize}
            {collisionDebug}
            on:close={() => (showDebug = false)}
        />
    {/if}
</div>
