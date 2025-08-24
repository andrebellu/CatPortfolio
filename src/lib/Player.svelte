<script>
  import { onMount, createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  const FRAME_WIDTH = 32;
  const FRAME_HEIGHT = 32;
  const MOVE_SPEED = 200;
  const FRAMES_PER_ANIM = 4;
  const IDLE_FRAME_DURATION = 700; // ms per frame idle
  const WALK_FRAME_DURATION = 150; // ms per frame walk

  export let x = 0;
  export let y = 0;
  export let debug = false;
  export let size = 96;

  let direction = "down";
  let playerEl;
  let keysPressed = new Set();
  let lastTime = performance.now();
  let isMoving = false;

  let currentFrame = 0;
  let frameTimer = 0;

  const idleAnimations = {
    down: { start: 0, row: 0 },
    downRight: { start: 4, row: 0 },
    right: { start: 8, row: 0 },
    upRight: { start: 12, row: 0 },
    up: { start: 16, row: 0 },
    upLeft: { start: 20, row: 0 },
    left: { start: 24, row: 0 },
    downLeft: { start: 28, row: 0 },
  };

  const walkAnimations = {
    down: { start: 0, row: 1 },
    downRight: { start: 4, row: 1 },
    right: { start: 8, row: 1 },
    upRight: { start: 12, row: 1 },
    up: { start: 16, row: 1 },
    upLeft: { start: 20, row: 1 },
    left: { start: 24, row: 1 },
    downLeft: { start: 28, row: 1 },
  };

  function updateFrame(dt) {
    frameTimer += dt;

    // scegli animazione
    const anim = isMoving
      ? walkAnimations[direction]
      : idleAnimations[direction];
    const frameDuration = isMoving ? WALK_FRAME_DURATION : IDLE_FRAME_DURATION;

    // avanza frame se è passato abbastanza tempo
    if (frameTimer >= frameDuration) {
      frameTimer = 0;
      currentFrame = (currentFrame + 1) % FRAMES_PER_ANIM;
    }

    // calcola indice frame
    const frameIndex = anim.start + currentFrame;

    // aggiorna posizione sprite
    if (playerEl) {
      playerEl.style.backgroundPositionX = -(frameIndex * FRAME_WIDTH) + "px";
      playerEl.style.backgroundPositionY = -(anim.row * FRAME_HEIGHT) + "px";
    }
  }

  onMount(() => {
    function gameLoop() {
      const now = performance.now();
      const dt = now - lastTime;
      lastTime = now;

      let dx = 0,
        dy = 0;
      if (keysPressed.has("arrowup") || keysPressed.has("w")) dy -= MOVE_SPEED;
      if (keysPressed.has("arrowdown") || keysPressed.has("s"))
        dy += MOVE_SPEED;
      if (keysPressed.has("arrowleft") || keysPressed.has("a"))
        dx -= MOVE_SPEED;
      if (keysPressed.has("arrowright") || keysPressed.has("d"))
        dx += MOVE_SPEED;

      isMoving = dx !== 0 || dy !== 0;

      if (isMoving && dx !== 0 && dy !== 0) {
        const norm = Math.SQRT1_2;
        dx *= norm;
        dy *= norm;
      }

      if (isMoving) {
        if (dy > 0 && dx === 0) direction = "down";
        else if (dy < 0 && dx === 0) direction = "up";
        else if (dx > 0 && dy === 0) direction = "right";
        else if (dx < 0 && dy === 0) direction = "left";
        else if (dx > 0 && dy > 0) direction = "downRight";
        else if (dx < 0 && dy > 0) direction = "downLeft";
        else if (dx > 0 && dy < 0) direction = "upRight";
        else if (dx < 0 && dy < 0) direction = "upLeft";
      }

      const newX = x + dx * (dt / 1000);
      const newY = y + dy * (dt / 1000);

      let cancelled = false;
      const cancel = () => (cancelled = true);
      dispatch("beforeMove", { newX, newY, cancel });

      if (!cancelled) {
        x = newX;
        y = newY;
      }

      updateFrame(dt);

      playerEl.style.transform = `translate(${x}px, ${y}px) scale(3)`;

      requestAnimationFrame(gameLoop);
    }

    requestAnimationFrame(gameLoop);

    const handleKey = (e, isDown) => {
      const k = e.key.toLowerCase();
      if (
        [
          "arrowup",
          "arrowdown",
          "arrowleft",
          "arrowright",
          "w",
          "a",
          "s",
          "d",
        ].includes(k)
      ) {
        e.preventDefault();
        isDown ? keysPressed.add(k) : keysPressed.delete(k);
      }
    };

    window.addEventListener("keydown", (e) => handleKey(e, true));
    window.addEventListener("keyup", (e) => handleKey(e, false));

    // Touch movement support
    let lastTouchDir = { dx: 0, dy: 0 };
    function handleTouchMovePlayer(e) {
      const { dx, dy } = e.detail;
      ["arrowup", "arrowdown", "arrowleft", "arrowright"].forEach((k) =>
        keysPressed.delete(k),
      );
      if (dy < 0) keysPressed.add("arrowup");
      if (dy > 0) keysPressed.add("arrowdown");
      if (dx < 0) keysPressed.add("arrowleft");
      if (dx > 0) keysPressed.add("arrowright");
      lastTouchDir = { dx, dy };
    }
    window.addEventListener("touchmoveplayer", handleTouchMovePlayer);
  });
</script>

<div
  bind:this={playerEl}
  class="absolute w-8 h-8 bg-no-repeat image-rendering-pixelated"
  style="background-image: url('src/assets/walk.png'); {debug
    ? 'outline: 2px solid red;'
    : ''}"
>
  {#if debug}
    <div
      class="absolute -top-6 left-0 text-xs bg-white px-1 border border-gray-300"
    >
      {Math.round(x)}, {Math.round(y)}
    </div>
  {/if}
</div>

<style>
  .image-rendering-pixelated {
    image-rendering: pixelated;
  }
</style>
