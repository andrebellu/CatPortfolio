<script>
  import { createEventDispatcher, onMount } from "svelte";
  const dispatch = createEventDispatcher();
  let startX = 0,
    startY = 0;
  let dx = 0,
    dy = 0;
  let active = false;
  let touchId = null;

  function handleTouchStart(e) {
    const t = e.touches[0];
    touchId = t.identifier;
    startX = t.clientX;
    startY = t.clientY;
    dx = 0;
    dy = 0;
    active = true;
  }
  function handleTouchMove(e) {
    if (!active) return;
    const t = Array.from(e.touches).find((t) => t.identifier === touchId);
    if (!t) return;
    dx = t.clientX - startX;
    dy = t.clientY - startY;
    let ndx = 0,
      ndy = 0;
    if (Math.abs(dx) > 20) ndx = dx > 0 ? 1 : -1;
    if (Math.abs(dy) > 20) ndy = dy > 0 ? 1 : -1;
    dispatch("move", { dx: ndx, dy: ndy });
  }
  function handleTouchEnd(e) {
    active = false;
    dx = 0;
    dy = 0;
    dispatch("move", { dx: 0, dy: 0 });
  }

  let isTouch = false;
  onMount(() => {
    isTouch = "ontouchstart" in window || navigator.maxTouchPoints > 0;
  });
  function handleEnterTap() {
    window.dispatchEvent(new KeyboardEvent("keydown", { key: "Enter" }));
    setTimeout(() => {
      window.dispatchEvent(new KeyboardEvent("keyup", { key: "Enter" }));
    }, 100);
  }
</script>

{#if isTouch}
  <div
    class="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 select-none flex flex-col items-center gap-4"
  >
    <div
      class="w-28 h-28 bg-white/30 rounded-full border-2 border-white flex items-center justify-center touch-none"
      ontouchstart={handleTouchStart}
      ontouchmove={handleTouchMove}
      ontouchend={handleTouchEnd}
      ontouchcancel={handleTouchEnd}
      style="backdrop-filter: blur(4px);"
    >
      <div
        class="w-12 h-12 bg-white/60 rounded-full border border-gray-400"
        style="transform: translate({dx / 2}px, {dy / 2}px);"
      ></div>
    </div>
    <button
      class="mt-2 px-6 py-2 rounded-lg bg-yellow-300 text-lg font-bold shadow border-2 border-yellow-500 active:scale-95 transition"
      ontouchstart={handleEnterTap}
      aria-label="Interagisci (ENTER)"
    >
      ENTER
    </button>
  </div>
{/if}
