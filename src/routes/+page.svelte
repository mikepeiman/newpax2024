<script>
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";

  let canvas;
  let ctx;
  const hexSize = 30;
  const gridSize = 15;

  onMount(() => {
    ctx = canvas.getContext("2d");
    drawHexGrid();
    drawStars();
  });

  function drawHexGrid() {
    for (let q = 0; q < gridSize; q++) {
      for (let r = 0; r < gridSize; r++) {
        drawHex(q, r);
      }
    }
  }

  function drawHex(q, r) {
    const x = hexSize * ((3 / 2) * q);
    const y = hexSize * ((Math.sqrt(3) / 2) * q + Math.sqrt(3) * r);
    ctx.beginPath();
    for (let i = 0; i < 6; i++) {
      const angle = ((2 * Math.PI) / 6) * i;
      const xi = x + hexSize * Math.cos(angle);
      const yi = y + hexSize * Math.sin(angle);
      if (i === 0) ctx.moveTo(xi, yi);
      else ctx.lineTo(xi, yi);
    }
    ctx.closePath();
    ctx.strokeStyle = "#00ffff";
    ctx.stroke();
  }

  function drawStars() {
    const starCount = 100;
    for (let i = 0; i < starCount; i++) {
      const x = Math.random() * canvas.width;
      const y = Math.random() * canvas.height;
      const radius = Math.random() * 2;
      ctx.beginPath();
      ctx.arc(x, y, radius, 0, 2 * Math.PI);
      ctx.fillStyle = "#ffffff";
      ctx.fill();
    }
  }

  function startGame() {
    goto("/game");
  }
</script>

<div class="game-container">
  <div class="top-menu">
    <h1>Stellar Conquest: Retro Edition</h1>
    <button on:click={startGame}>Start Your Conquest</button>
  </div>
  <div class="right-menu">
    <p>
      Embark on an epic journey through abstract star systems in this tick-based
      strategy game. Conquer planets, manage resources, and outsmart your
      opponents in a stylized retro universe.
    </p>
  </div>
  <canvas bind:this={canvas} width="800" height="800"></canvas>
</div>

<style>
  .game-container {
    display: grid;
    grid-template-columns: 1fr 20%;
    grid-template-rows: auto 1fr;
    height: 100vh;
    font-family: "Courier New", monospace;
    background-color: #000;
    color: #00ffff;
  }

  .top-menu {
    grid-column: 1 / -1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    background-color: rgba(0, 0, 0, 0.8);
  }

  .right-menu {
    grid-column: 2;
    grid-row: 2;
    padding: 1rem;
    background-color: rgba(0, 0, 0, 0.8);
    overflow-y: auto;
  }

  canvas {
    grid-column: 1;
    grid-row: 2;
    width: 100%;
    height: 100%;
  }

  h1 {
    font-size: 1.5rem;
    margin: 0;
    text-shadow: 0 0 10px #00ffff;
  }

  p {
    font-size: 0.9rem;
    margin-bottom: 1rem;
  }

  button {
    padding: 0.5rem 1rem;
    font-size: 1rem;
    background-color: #ff00ff;
    color: #000;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  button:hover {
    background-color: #00ffff;
    color: #000;
    transform: scale(1.1);
  }
</style>
