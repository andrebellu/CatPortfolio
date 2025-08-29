<script>
    import { createEventDispatcher, onMount } from "svelte";
    export let showDebug = false;
    const dispatch = createEventDispatcher();

    const menuTextsIta = {
        lang: "ita",
        debug: "Visualizza debug",
        tech: "Tecnologie utilizzate",
        controls: "Controlli",
        opt: "Lingua",
    };

    const menuTextsEng = {
        lang: "eng",
        debug: "Show debug",
        tech: "Technologies used",
        controls: "Controls",
        opt: "Language",
    };

    let currentMenuTexts = menuTextsIta;

    let drawerOpen = false;
    let isMobile = false;
    function toggleDrawer() {
        drawerOpen = !drawerOpen;
    }
    onMount(() => {
        const check = () => {
            isMobile = window.innerWidth < 768;
            if (!isMobile) drawerOpen = false;
        };
        check();
        window.addEventListener("resize", check);
        return () => window.removeEventListener("resize", check);
    });
</script>

<button
    class="md:hidden fixed top-4 left-4 z-50 w-12 h-12 flex flex-col justify-center items-center rounded-full border shadow-lg"
    on:click={toggleDrawer}
    aria-label="Apri menu"
>
    <span class="block w-7 h-1 bg-gray-700 rounded mb-1"></span>
    <span class="block w-7 h-1 bg-gray-700 rounded mb-1"></span>
    <span class="block w-7 h-1 bg-gray-700 rounded"></span>
</button>

{#if isMobile}
    <div
        class="fixed inset-0 bg-black/40 z-40 transition-opacity duration-300"
        style:opacity={drawerOpen ? 1 : 0}
        style:pointer-events={drawerOpen ? "auto" : "none"}
        on:click={toggleDrawer}
    ></div>
{/if}

<div
    class="p-6 bg-white/90 h-full w-[80vw] max-w-xs fixed md:static top-0 left-0 z-50 shadow-2xl border-r border-gray-300 transition-transform duration-300 md:w-full md:max-w-full md:h-auto md:shadow-none md:bg-transparent"
    class:translate-x-0={drawerOpen || !isMobile}
    class:-translate-x-full={!drawerOpen && isMobile}
    style="will-change: transform;"
>
    <h2 class="font-semibold mb-4 text-6xl text-center">Menu</h2>
    <ul class="space-y-2 text-4xl">
        <li>
            <button
                class="w-full hover:underline text-center"
                on:click={() => {
                    dispatch("toggleDebug");
                    if (isMobile) drawerOpen = false;
                }}
            >
                {currentMenuTexts.debug}
            </button>
        </li>
        <li>
            <button class="w-full text-center hover:underline">
                {currentMenuTexts.tech}
            </button>
        </li>
        <li>
            <button class="w-full text-center hover:underline">
                {currentMenuTexts.controls}
            </button>
        </li>
        <li>
            <button
                class="w-full text-center hover:underline"
                on:click={() =>
                    (currentMenuTexts =
                        currentMenuTexts.lang === "ita"
                            ? menuTextsEng
                            : menuTextsIta)}
            >
                {currentMenuTexts.opt}
            </button>
        </li>
    </ul>
</div>
