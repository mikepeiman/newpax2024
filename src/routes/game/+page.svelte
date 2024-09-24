<script>
  import { onMount } from "svelte";

  let gameState = {
    turn: 1,
    resources: 100,
    selectedPlanet: null,
    gridSize: { width: 10, height: 10 }, // Rectangular grid size
    hexSize: 50, // Default hex size in pixels
    staggered: true, // Enable staggered rows
    activeStar: null,
    lastActiveStar: null,
    stars: [],
    paths: [],
  };

  let canvas;
  let ctx;
  let gameContainer;
  const MAX_HEX_SIZE = 50;

  let isDragging = false;
  let lastMousePosition = { x: 0, y: 0 };

  onMount(() => {
    canvas = document.getElementById("gameCanvas");
    ctx = canvas.getContext("2d");
    gameContainer = document.querySelector(".game-container");

    canvas.addEventListener("mousedown", handleCanvasMouseDown);
    canvas.addEventListener("mousemove", handleCanvasMouseMove);
    canvas.addEventListener("mouseup", handleCanvasMouseUp);
    canvas.addEventListener("mouseleave", handleCanvasMouseUp);

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);

    return () => {
      window.removeEventListener("resize", resizeCanvas);
      canvas.removeEventListener("mousedown", handleCanvasMouseDown);
      canvas.removeEventListener("mousemove", handleCanvasMouseMove);
      canvas.removeEventListener("mouseup", handleCanvasMouseUp);
      canvas.removeEventListener("mouseleave", handleCanvasMouseUp);
    };
  });

  function resizeCanvas() {
    if (!gameContainer || !canvas) return;

    const containerWidth = gameContainer.clientWidth;
    const containerHeight = gameContainer.clientHeight;

    canvas.width = containerWidth;
    canvas.height = containerHeight;

    const widthHexSize =
      containerWidth / ((gameState.gridSize.width - 0.5) * Math.sqrt(3));
    const heightHexSize =
      containerHeight / ((gameState.gridSize.height - 0.25) * 1.5);

    gameState.hexSize = Math.min(widthHexSize, heightHexSize, MAX_HEX_SIZE);

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
      { color: "#ffffff", radius: 15, pulseSpeed: 0.5 },
      { color: "#ffcc00", radius: 20, pulseSpeed: 0.7 },
      { color: "#ff6699", radius: 18, pulseSpeed: 0.6 },
      { color: "#66ccff", radius: 12, pulseSpeed: 0.8 },
      { color: "#ff33cc", radius: 16, pulseSpeed: 0.9 },
    ];

    let availableHexes = [];

    for (let r = 0; r < gameState.gridSize.height; r++) {
      for (let q = 0; q < gameState.gridSize.width; q++) {
        availableHexes.push({ q, r });
      }
    }

    const offsetX =
      (canvas.width -
        (gameState.gridSize.width - 1) * gameState.hexSize * Math.sqrt(3)) /
      2;
    const offsetY =
      (canvas.height -
        (gameState.gridSize.height - 1) * gameState.hexSize * 1.5) /
      2;

    gameState.stars = []; // Reset stars array

    for (let i = 0; i < starCount; i++) {
      if (availableHexes.length === 0) break;
      const index = Math.floor(Math.random() * availableHexes.length);
      const { q, r } = availableHexes.splice(index, 1)[0];

      const xOffset = r % 2 === 0 ? 0 : gameState.hexSize * (Math.sqrt(3) / 2);
      const x = offsetX + gameState.hexSize * (Math.sqrt(3) * q) + xOffset;
      const y = offsetY + gameState.hexSize * (1.5 * r);

      const starType = starTypes[Math.floor(Math.random() * starTypes.length)];
      gameState.stars.push({
        id: i + 1,
        x,
        y,
        q,
        r,
        ...starType,
        phase: Math.random() * Math.PI * 2,
      });
    }

    // Draw paths
    ctx.lineWidth = 3;
    ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
    gameState.paths.forEach((path) => {
      ctx.beginPath();
      ctx.moveTo(path.start.x, path.start.y);
      ctx.lineTo(path.end.x, path.end.y);
      ctx.stroke();
    });

    // Draw stars
    gameState.stars.forEach((star) => {
      const pulseScale = 0.2 * Math.sin(star.phase) + 1;
      const radius = star.radius * pulseScale;

      const gradient = ctx.createRadialGradient(
        star.x,
        star.y,
        0,
        star.x,
        star.y,
        radius * 2,
      );
      gradient.addColorStop(0, star.color);
      gradient.addColorStop(0.7, star.color + "80");
      gradient.addColorStop(1, "transparent");

      ctx.beginPath();
      ctx.arc(star.x, star.y, radius * 2, 0, Math.PI * 2);
      ctx.fillStyle = gradient;
      ctx.fill();

      ctx.beginPath();
      ctx.arc(star.x, star.y, radius * 0.5, 0, Math.PI * 2);
      ctx.fillStyle = "white";
      ctx.fill();

      star.phase += star.pulseSpeed * 0.015;
    });

    requestAnimationFrame(drawStars);
  }

  function getStarAtPosition(x, y) {
    const q = (x - gameState.hexSize) / (gameState.hexSize * Math.sqrt(3));
    const r = (y - gameState.hexSize) / (gameState.hexSize * 1.5);
    const roundedQ = Math.round(q);
    const roundedR = Math.round(r);
    return gameState.stars.find(
      (star) => star.q === roundedQ && star.r === roundedR,
    );
  }

  function addPath(startStar, endStar) {
    if (startStar && endStar && startStar !== endStar) {
      const existingPath = gameState.paths.find(
        (path) =>
          (path.start === startStar && path.end === endStar) ||
          (path.start === endStar && path.end === startStar),
      );
      if (!existingPath) {
        gameState.paths.push({
          start: { x: startStar.x, y: startStar.y },
          end: { x: endStar.x, y: endStar.y },
        });
      }
    }
  }

  function updateStarSelection(star) {
    if (star) {
      gameState.lastActiveStar = gameState.activeStar;
      gameState.activeStar = star.id;
      console.log(
        `Active star: ${gameState.activeStar}, Last star: ${gameState.lastActiveStar}`,
      );
    }
  }

  function handleCanvasMouseDown(event) {
    isDragging = true;
    const rect = canvas.getBoundingClientRect();
    lastMousePosition.x = event.clientX - rect.left;
    lastMousePosition.y = event.clientY - rect.top;
    const clickedStar = getStarAtPosition(
      lastMousePosition.x,
      lastMousePosition.y,
    );
    updateStarSelection(clickedStar);
    drawGameBoard();
  }

  function handleCanvasMouseMove(event) {
    if (!isDragging) return;

    const rect = canvas.getBoundingClientRect();
    const currentX = event.clientX - rect.left;
    const currentY = event.clientY - rect.top;

    const startStar = getStarAtPosition(
      lastMousePosition.x,
      lastMousePosition.y,
    );
    const endStar = getStarAtPosition(currentX, currentY);

    if (endStar && endStar !== startStar) {
      addPath(startStar, endStar);
      updateStarSelection(endStar);
      lastMousePosition.x = currentX;
      lastMousePosition.y = currentY;
      drawGameBoard();
    }
  }

  function handleCanvasMouseUp() {
    isDragging = false;
  }

  function adjustGridSize(size) {
    gameState.gridSize.width = size;
    gameState.gridSize.height = size;
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
      <p class="stat">
        Active Star: {gameState.activeStar ? gameState.activeStar : "None"}
      </p>
      <p class="stat">
        Last Active Star: {gameState.lastActiveStar
          ? gameState.lastActiveStar
          : "None"}
      </p>
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

<!-- <script>
  import { onMount } from "svelte";

  let gameState = {
    turn: 1,
    resources: 100,
    selectedPlanet: null,
    gridSize: { width: 10, height: 10 }, // Rectangular grid size
    hexSize: 50, // Default hex size in pixels
    staggered: true, // Enable staggered rows
    activeStar: null,
    lastActiveStar: null,
    stars: [],
  };

  let canvas;
  let ctx;
  let gameContainer;
  const MAX_HEX_SIZE = 50;

  onMount(() => {
    canvas = document.getElementById("gameCanvas");
    ctx = canvas.getContext("2d");
    gameContainer = document.querySelector(".game-container");

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);
    canvas.addEventListener("mousedown", handleMouseDown);

    return () => {
      window.removeEventListener("resize", resizeCanvas);
      canvas.removeEventListener("mousedown", handleMouseDown);
    };
  });

  function resizeCanvas() {
    if (!gameContainer || !canvas) return;

    const containerWidth = gameContainer.clientWidth;
    const containerHeight = gameContainer.clientHeight;

    canvas.width = containerWidth;
    canvas.height = containerHeight;

    const widthHexSize =
      containerWidth / ((gameState.gridSize.width - 0.5) * Math.sqrt(3));
    const heightHexSize =
      containerHeight / ((gameState.gridSize.height - 0.25) * 1.5);

    gameState.hexSize = Math.min(widthHexSize, heightHexSize, MAX_HEX_SIZE);

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
      { color: "#ffffff", radius: 15, pulseSpeed: 0.5 },
      { color: "#ffcc00", radius: 20, pulseSpeed: 0.7 },
      { color: "#ff6699", radius: 18, pulseSpeed: 0.6 },
      { color: "#66ccff", radius: 12, pulseSpeed: 0.8 },
      { color: "#ff33cc", radius: 16, pulseSpeed: 0.9 },
    ];

    let availableHexes = [];

    for (let r = 0; r < gameState.gridSize.height; r++) {
      for (let q = 0; q < gameState.gridSize.width; q++) {
        availableHexes.push({ q, r });
      }
    }

    const offsetX =
      (canvas.width -
        (gameState.gridSize.width - 1) * gameState.hexSize * Math.sqrt(3)) /
      2;
    const offsetY =
      (canvas.height -
        (gameState.gridSize.height - 1) * gameState.hexSize * 1.5) /
      2;

    gameState.stars = []; // Reset stars array

    for (let i = 0; i < starCount; i++) {
      if (availableHexes.length === 0) break;
      const index = Math.floor(Math.random() * availableHexes.length);
      const { q, r } = availableHexes.splice(index, 1)[0];

      const xOffset = r % 2 === 0 ? 0 : gameState.hexSize * (Math.sqrt(3) / 2);
      const x = offsetX + gameState.hexSize * (Math.sqrt(3) * q) + xOffset;
      const y = offsetY + gameState.hexSize * (1.5 * r);

      const starType = starTypes[Math.floor(Math.random() * starTypes.length)];
      gameState.stars.push({
        id: i + 1,
        x,
        y,
        ...starType,
        phase: Math.random() * Math.PI * 2,
      });
    }

    animateStars();
  }

  function handleMouseDown(event) {
    const rect = canvas.getBoundingClientRect();
    const mouseX = event.clientX - rect.left;
    const mouseY = event.clientY - rect.top;

    if (event.button === 0) {
      // Left click
      const clickedStar = gameState.stars.find((star) =>
        isPointInStar(mouseX, mouseY, star),
      );
      if (clickedStar) {
        gameState.lastActiveStar = gameState.activeStar;
        gameState.activeStar = clickedStar;
      }
    } else if (event.button === 2) {
      // Right click
      gameState.activeStar = null;
    }
  }

  function isPointInStar(mouseX, mouseY, star) {
    const pulseScale = 0.2 * Math.sin(star.phase) + 1;
    const radius = star.radius * pulseScale * 2; // Adjust for visual size
    return Math.hypot(mouseX - star.x, mouseY - star.y) <= radius;
  }

  function animateStars() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawHexGrid();

    gameState.stars.forEach((star) => {
      const pulseScale = 0.2 * Math.sin(star.phase) + 1;
      const radius = star.radius * pulseScale;

      const gradient = ctx.createRadialGradient(
        star.x,
        star.y,
        0,
        star.x,
        star.y,
        radius * 2,
      );
      gradient.addColorStop(0, star.color);
      gradient.addColorStop(0.7, star.color + "80");
      gradient.addColorStop(1, "transparent");

      ctx.beginPath();
      ctx.arc(star.x, star.y, radius * 2, 0, Math.PI * 2);
      ctx.fillStyle = gradient;
      ctx.fill();

      ctx.beginPath();
      ctx.arc(star.x, star.y, radius * 0.5, 0, Math.PI * 2);
      ctx.fillStyle = "white";
      ctx.fill();

      star.phase += star.pulseSpeed * 0.015;
    });

    requestAnimationFrame(animateStars);
  }

  function adjustGridSize(size) {
    gameState.gridSize.width = size;
    gameState.gridSize.height = size;
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
      <p class="stat">
        Active Star: {gameState.activeStar ? gameState.activeStar.id : "None"}
      </p>
      <p class="stat">
        Last Active Star: {gameState.lastActiveStar
          ? gameState.lastActiveStar.id
          : "None"}
      </p>
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
-->
<style>
  .game-container {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  canvas {
    width: 100%;
    height: 100%;
    box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
    border-radius: 10px;
    background: radial-gradient(ellipse at center, #0a2e38 0%, #000000 70%);
  }

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

  .game-container {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  canvas {
    width: 100%;
    height: 100%;
    box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
    border-radius: 10px;
    background: radial-gradient(ellipse at center, #0a2e38 0%, #000000 70%);
  }

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
</style>
