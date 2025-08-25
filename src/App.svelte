<script>
  import Map from "./lib/Map.svelte";
  import Menu from "./lib/Menu.svelte";
  import TouchMovement from "./lib/TouchMovement.svelte";
  import "./app.css";

  let showDebug = false;

  function handleMove(e) {
    window.dispatchEvent(
      new CustomEvent("touchmoveplayer", { detail: e.detail }),
    );
  }
</script>

<div class="flex w-full h-screen overflow-hidden">
  <div class="w-1/4 bg-gray-100 border-r border-gray-300 overflow-y-auto">
    <Menu {showDebug} on:toggleDebug={() => (showDebug = !showDebug)} />
  </div>

  <div class="flex-1 relative bg-gray-200">
    <Map {showDebug} />
  </div>
</div>

<TouchMovement on:move={handleMove} />
