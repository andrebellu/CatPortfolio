<script>
  import Map from "./lib/Map.svelte";
  import TouchMovement from "./lib/TouchMovement.svelte";
  import "./app.css";

  function handleMove(e) {
    window.dispatchEvent(
      new CustomEvent("touchmoveplayer", { detail: e.detail }),
    );
  }
</script>

<Map />
<TouchMovement on:move={handleMove} />
