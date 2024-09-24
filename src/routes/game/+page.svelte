<script>
  import { onMount } from "svelte";

  let gameState = {
    turn: 1,
    resources: 100,
    selectedPlanet: null,
    gridSize: { width: 10, height: 10 }, // Rectangular grid size
    hexSize: 50, // Default hex size in pixels
    staggered: true, // Enable staggered rows
  };

  let canvas;
  let ctx;

  onMount(() => {
    canvas = document.getElementById("gameCanvas");
    ctx = canvas.getContext("2d");
    resizeCanvas();
    drawGameBoard();
    window.addEventListener("resize", resizeCanvas);
  });

  function resizeCanvas() {
    const gameContainer = document.querySelector(".game-container");
    const size =
      Math.min(gameContainer.clientWidth, gameContainer.clientHeight) * 0.8;
    canvas.width = size;
    canvas.height = size;
    drawGameBoard();
  }

  function endTurn() {
    gameState.turn++;
    // Add logic for end of turn actions
  }

  function selectPlanet(planet) {
    gameState.selectedPlanet = planet;
    // Add logic for planet selection
  }

  function drawGameBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawHexGrid();
    drawStars();
  }

  function drawHexGrid() {
    const offsetX =
      (canvas.width -
        (gameState.gridSize.width - 1) * gameState.hexSize * Math.sqrt(3)) /
      2;
    const offsetY =
      (canvas.height -
        (gameState.gridSize.height - 1) * gameState.hexSize * 1.5) /
      2;

    for (let r = 0; r < gameState.gridSize.height; r++) {
      for (let q = 0; q < gameState.gridSize.width; q++) {
        // Stagger every other row
        const xOffset =
          r % 2 === 0 ? 0 : gameState.hexSize * (Math.sqrt(3) / 2);
        drawHex(q, r, offsetX + xOffset, offsetY);
      }
    }
  }

  function drawHex(q, r, offsetX, offsetY) {
    const x = offsetX + gameState.hexSize * (Math.sqrt(3) * q);
    const y = offsetY + gameState.hexSize * (1.5 * r);
    ctx.beginPath();
    for (let i = 0; i < 6; i++) {
      const angle = ((2 * Math.PI) / 6) * i + Math.PI / 6;
      const xi = x + gameState.hexSize * Math.cos(angle);
      const yi = y + gameState.hexSize * Math.sin(angle);
      if (i === 0) ctx.moveTo(xi, yi);
      else ctx.lineTo(xi, yi);
    }
    ctx.closePath();
    ctx.strokeStyle = "#00ffff";
    ctx.stroke();
  }

  function drawStars() {
    const starCount = 30;
    const starTypes = [
      { color: "#ffffff", radius: 1.5, pulseSpeed: 0.5 },
      { color: "#ffcc00", radius: 2, pulseSpeed: 0.7 },
      { color: "#ff6699", radius: 1.8, pulseSpeed: 0.6 },
      { color: "#66ccff", radius: 2.2, pulseSpeed: 0.8 },
      { color: "#ff33cc", radius: 2, pulseSpeed: 0.9 },
    ];

    let stars = [];
    let availableHexes = [];

    // Generate all possible hex positions
    for (let r = 0; r < gameState.gridSize.height; r++) {
      for (let q = 0; q < gameState.gridSize.width; q++) {
        availableHexes.push({ q, r });
      }
    }

    // Randomly select 30 hexes for stars
    for (let i = 0; i < starCount; i++) {
      if (availableHexes.length === 0) break;
      const index = Math.floor(Math.random() * availableHexes.length);
      const { q, r } = availableHexes.splice(index, 1)[0];

      const offsetX =
        (canvas.width -
          (gameState.gridSize.width - 1) * gameState.hexSize * Math.sqrt(3)) /
        2;
      const offsetY =
        (canvas.height -
          (gameState.gridSize.height - 1) * gameState.hexSize * 1.5) /
        2;

      // Calculate the center of the hex
      const x =
        offsetX +
        gameState.hexSize * (Math.sqrt(3) * q + (Math.sqrt(3) / 2) * r);
      const y = offsetY + gameState.hexSize * (1.5 * r);

      const starType = starTypes[Math.floor(Math.random() * starTypes.length)];
      stars.push({ x, y, ...starType, phase: Math.random() * Math.PI * 2 });
    }

    function animateStars() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawHexGrid();

      stars.forEach((star) => {
        const pulseScale = 0.2 * Math.sin(star.phase) + 1;
        const radius = star.radius * pulseScale;

        // Create radial gradient for the star
        const gradient = ctx.createRadialGradient(
          star.x,
          star.y,
          0,
          star.x,
          star.y,
          radius * 2,
        );
        gradient.addColorStop(0, star.color);
        gradient.addColorStop(0.7, star.color + "80"); // 50% opacity
        gradient.addColorStop(1, "transparent");

        ctx.beginPath();
        ctx.arc(star.x, star.y, radius * 2, 0, Math.PI * 2);
        ctx.fillStyle = gradient;
        ctx.fill();

        // Add a bright center to the star
        ctx.beginPath();
        ctx.arc(star.x, star.y, radius * 0.5, 0, Math.PI * 2);
        ctx.fillStyle = "white";
        ctx.fill();

        // Update the star's phase for the next frame
        star.phase += star.pulseSpeed * 0.05;
      });

      requestAnimationFrame(animateStars);
    }

    animateStars();
  }

  function adjustGridSize(size) {
    gameState.gridSize.width = size;
    gameState.gridSize.height = size; // Adjust height as well
    drawGameBoard();
  }

  function adjustHexSize(size) {
    gameState.hexSize = size;
    drawGameBoard();
  }
</script>

<div class="game-container">
  <header class="game-header">
    <h2 class="game-title">Stellar Conquest: Game Area</h2>
    <div class="game-stats">
      <p class="stat">Turn: {gameState.turn}</p>
      <button on:click={endTurn} class="btn btn-primary">End Turn</button>
      <p class="stat">Resources: {gameState.resources}</p>
    </div>
  </header>

  <main class="game-main">
    <canvas id="gameCanvas" class="game-canvas"></canvas>
  </main>

  <aside class="game-sidebar">
    <section class="sidebar-section">
      <h3 class="section-title">Settings</h3>
      <div class="setting">
        <label for="gridSize" class="setting-label">Grid Size:</label>
        <input
          type="range"
          id="gridSize"
          min="5"
          max="50"
          bind:value={gameState.gridSize.width}
          on:change={() => adjustGridSize(gameState.gridSize.width)}
          class="setting-input" />
        <span class="setting-value"
          >{gameState.gridSize.width}x{gameState.gridSize.height}</span>
      </div>
      <div class="setting">
        <label for="hexSize" class="setting-label">Hex Size:</label>
        <input
          type="range"
          id="hexSize"
          min="30"
          max="100"
          bind:value={gameState.hexSize}
          on:change={() => adjustHexSize(gameState.hexSize)}
          class="setting-input" />
        <span class="setting-value">{gameState.hexSize}px</span>
      </div>
    </section>

    <section class="sidebar-section">
      <h3 class="section-title">Game Actions</h3>
      <button class="btn btn-secondary">Sound On/Off</button>
      <button class="btn btn-secondary">Save Game</button>
      <button class="btn btn-secondary">Exit Game</button>
      <button class="btn btn-secondary">Build</button>
      <button class="btn btn-secondary">Research</button>
    </section>

    <section class="sidebar-section">
      <h3 class="section-title">Planet Info</h3>
      {#if gameState.selectedPlanet}
        <p class="planet-info">Selected Planet: {gameState.selectedPlanet}</p>
      {:else}
        <p class="planet-info">No planet selected</p>
      {/if}
    </section>
  </aside>
</div>

<style>
  .game-container {
    display: grid;
    grid-template-columns: 1fr 300px;
    grid-template-rows: auto 1fr;
    height: 100vh;
    background-color: #0a0a0a;
    color: #e0e0e0;
    font-family: "Arial", sans-serif;
  }

  .game-header {
    grid-column: 1 / -1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    background-color: #1a1a1a;
    border-bottom: 1px solid #2a2a2a;
  }

  .game-title {
    font-size: 1.5rem;
    color: #00ffff;
    margin: 0;
  }

  .game-stats {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .stat {
    font-size: 1rem;
    margin: 0;
  }

  .game-main {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 2rem;
  }

  .game-canvas {
    border: 1px solid #2a2a2a;
    background-color: #000000;
  }

  .game-sidebar {
    padding: 2rem;
    background-color: #1a1a1a;
    border-left: 1px solid #2a2a2a;
    overflow-y: auto;
  }

  .sidebar-section {
    margin-bottom: 2rem;
  }

  .section-title {
    font-size: 1.2rem;
    color: #00ffff;
    margin-bottom: 1rem;
  }

  .setting {
    margin-bottom: 1rem;
  }

  .setting-label {
    display: block;
    margin-bottom: 0.5rem;
  }

  .setting-input {
    width: 100%;
    margin-bottom: 0.5rem;
  }

  .setting-value {
    font-size: 0.9rem;
    color: #00ffff;
  }

  .btn {
    display: block;
    width: 100%;
    padding: 0.75rem;
    margin-bottom: 0.5rem;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .btn-primary {
    background-color: #00ffff;
    color: #000000;
  }

  .btn-primary:hover {
    background-color: #00cccc;
  }

  .btn-secondary {
    background-color: #2a2a2a;
    color: #e0e0e0;
  }

  .btn-secondary:hover {
    background-color: #3a3a3a;
  }

  .planet-info {
    font-size: 0.9rem;
  }

  canvas {
    box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
    border-radius: 10px;
    background: radial-gradient(ellipse at center, #0a2e38 0%, #000000 70%);
  }
</style>
