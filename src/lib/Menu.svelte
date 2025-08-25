<script>
  import { createEventDispatcher } from "svelte";
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
</script>

<div class="p-6">
  <h2 class="font-semibold mb-4 text-6xl">Menu</h2>
  <ul class="space-y-2 text-4xl">
    <li>
      <button
        class="w-full text-left hover:underline"
        on:click={() => dispatch("toggleDebug")}
      >
        {currentMenuTexts.debug}
      </button>
    </li>
    <li>
      <button class="w-full text-left hover:underline">
        {currentMenuTexts.tech}
      </button>
    </li>
    <li>
      <button class="w-full text-left hover:underline">
        {currentMenuTexts.controls}
      </button>
    </li>
    <li>
      <button
        class="w-full text-left hover:underline"
        on:click={() =>
          (currentMenuTexts =
            currentMenuTexts.lang === "ita" ? menuTextsEng : menuTextsIta)}
      >
        {currentMenuTexts.opt}
      </button>
    </li>
  </ul>
</div>
