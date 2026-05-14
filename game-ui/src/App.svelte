<script>
  // @ts-nocheck

  import { onMount } from "svelte";
  import PlayerHud from "./components/PlayerHud.svelte";
  import Inventory from "./components/Inventory.svelte";
  import GameOver from "./components/GameOver.svelte";

  import meteorImg from "./assets/meteor.png";

  import backgroundMusic from "./assets/sounds/background.mp3";
  import shootSound from "./assets/sounds/shoot.mp3";
  import enemyDeathSound from "./assets/sounds/enemy-death.mp3";
  import damageSound from "./assets/sounds/damage.mp3";
  import gameOverSound from "./assets/sounds/gameover.mp3";

  let gameArea;

  let gameStarted = false;
  let gameOver = false;

  let mouseX = 400;
  let mouseY = 500;

  let lives = 3;
  let score = 0;
  let highscore = 0;

  let fireRate = 450;
  let lastShot = 0;
  let rapidFireActive = false;

  let difficulty = 1;
  let maxEnemies = 4;

  let bullets = [];
  let enemyBullets = [];
  let enemies = [];
  let powerups = [];
  let obstacles = [];
  let inventory = [];
  let effects = [];

  let enemyId = 1;
  let bulletId = 1;
  let powerupId = 1;
  let effectId = 1;
  let nextPowerupType = "Rapid Fire";

  let musicAudio;
  let shootAudio;
  let enemyDeathAudio;
  let damageAudio;
  let gameOverAudio;
  let soundStarted = false;

  onMount(() => {
    highscore = Number(localStorage.getItem("spaceHighscore")) || 0;
    createObstacles();

    musicAudio = new Audio(backgroundMusic);
    musicAudio.loop = true;
    musicAudio.volume = 0.25;

    musicAudio.addEventListener("ended", () => {
      musicAudio.currentTime = 0.05;
      musicAudio.play().catch(() => {});
    });

    shootAudio = new Audio(shootSound);
    shootAudio.volume = 0.25;

    enemyDeathAudio = new Audio(enemyDeathSound);
    enemyDeathAudio.volume = 0.5;

    damageAudio = new Audio(damageSound);
    damageAudio.volume = 0.6;

    gameOverAudio = new Audio(gameOverSound);
    gameOverAudio.volume = 0.7;

    const loop = setInterval(updateGame, 30);
    const spawnEnemies = setInterval(spawnEnemy, 1500);
    const spawnPowerups = setInterval(spawnPowerup, 10000);
    const enemyShoot = setInterval(enemyFire, 1500);
    const increaseDifficulty = setInterval(makeHarder, 12000);
    const autoShoot = setInterval(shoot, 50);

    window.addEventListener("keydown", handleKeyPress);
    window.addEventListener("click", startSound);
    window.addEventListener("touchstart", startSound);

    return () => {
      clearInterval(loop);
      clearInterval(spawnEnemies);
      clearInterval(spawnPowerups);
      clearInterval(enemyShoot);
      clearInterval(increaseDifficulty);
      clearInterval(autoShoot);

      window.removeEventListener("keydown", handleKeyPress);
      window.removeEventListener("click", startSound);
      window.removeEventListener("touchstart", startSound);

      if (musicAudio) musicAudio.pause();
    };
  });

  function getGameWidth() {
    return gameArea ? gameArea.clientWidth : 1000;
  }

  function getGameHeight() {
    return gameArea ? gameArea.clientHeight : 620;
  }

  function startGame() {
    gameStarted = true;
    startSound();

    mouseX = getGameWidth() / 2;
    mouseY = getGameHeight() - 120;
  }

  function startSound() {
    if (soundStarted) return;
    if (!musicAudio) return;

    soundStarted = true;
    musicAudio.currentTime = 0;
    musicAudio.play().catch(() => {});
  }

  function playSound(audio) {
    if (!audio) return;

    audio.currentTime = 0;
    audio.play().catch(() => {});
  }

  function createObstacles() {
    obstacles = [
      { id: 1, x: 220, y: -100, w: 70, h: 70, speed: 2.2, rotation: 0 },
      { id: 2, x: 620, y: -350, w: 85, h: 85, speed: 1.8, rotation: 40 },
      { id: 3, x: 850, y: -600, w: 75, h: 75, speed: 2, rotation: 90 },
    ];
  }

  function makeHarder() {
    if (!gameStarted || gameOver) return;

    difficulty += 0.2;

    if (maxEnemies < 8) {
      maxEnemies += 1;
    }
  }

  function movePlayer(event) {
    if (!gameStarted || gameOver) return;

    startSound();

    const rect = gameArea.getBoundingClientRect();
    mouseX = event.clientX - rect.left;
    mouseY = event.clientY - rect.top;
  }

  function movePlayerTouch(event) {
    if (!gameStarted || gameOver) return;

    startSound();

    const touch = event.touches[0];
    const rect = gameArea.getBoundingClientRect();

    mouseX = touch.clientX - rect.left;
    mouseY = touch.clientY - rect.top;
  }

  function shoot() {
    if (!gameStarted || gameOver) return;

    const now = Date.now();

    if (now - lastShot < fireRate) return;

    lastShot = now;

    bullets = [
      ...bullets,
      {
        id: bulletId++,
        x: mouseX + 15,
        y: mouseY,
        speed: 14,
      },
    ];

    playSound(shootAudio);
    createEffect(mouseX + 15, mouseY - 20, "muzzle");
  }

  function spawnEnemy() {
    if (!gameStarted || gameOver) return;
    if (enemies.length >= maxEnemies) return;

    const gameWidth = getGameWidth();

    enemies = [
      ...enemies,
      {
        id: enemyId++,
        x: Math.random() * (gameWidth - 100) + 50,
        y: -60,
        speed: Math.random() * 1.5 + difficulty,
      },
    ];
  }

  function spawnPowerup() {
    if (!gameStarted || gameOver) return;
    if (powerups.length >= 1) return;

    const gameWidth = getGameWidth();
    const gameHeight = getGameHeight();

    const type = nextPowerupType;
    nextPowerupType = nextPowerupType === "Rapid Fire" ? "Life" : "Rapid Fire";

    powerups = [
      ...powerups,
      {
        id: powerupId++,
        x: Math.random() * (gameWidth - 100) + 50,
        y: Math.random() * (gameHeight - 180) + 80,
        type,
      },
    ];
  }

  function enemyFire() {
    if (!gameStarted || gameOver) return;

    enemies.forEach((enemy) => {
      enemyBullets = [
        ...enemyBullets,
        {
          id: bulletId++,
          x: enemy.x + 22,
          y: enemy.y + 45,
          speed: 6,
        },
      ];
    });
  }

  function updateGame() {
    if (!gameStarted || gameOver) return;

    const gameWidth = getGameWidth();
    const gameHeight = getGameHeight();

    bullets = bullets
      .map((b) => ({ ...b, y: b.y - b.speed }))
      .filter((b) => b.y > -20);

    enemyBullets = enemyBullets
      .map((b) => ({ ...b, y: b.y + b.speed }))
      .filter((b) => b.y < gameHeight + 50);

    enemies = enemies
      .map((e) => ({ ...e, y: e.y + e.speed }))
      .filter((e) => e.y < gameHeight + 50);

    obstacles = obstacles.map((o) => {
      const newY = o.y + o.speed + difficulty * 0.2;

      if (newY > gameHeight + 80) {
        return {
          ...o,
          x: Math.random() * (gameWidth - 80) + 40,
          y: -Math.random() * 500 - 100,
          speed: Math.random() * 1.5 + 1.5,
          rotation: Math.random() * 360,
        };
      }

      return {
        ...o,
        y: newY,
        rotation: (o.rotation || 0) + 3,
      };
    });

    effects = effects
      .map((effect) => ({ ...effect, life: effect.life - 1 }))
      .filter((effect) => effect.life > 0);

    checkCollisions();
  }

  function checkCollisions() {
    bullets.forEach((bullet) => {
      enemies.forEach((enemy) => {
        if (isColliding(bullet, enemy, 8, 8, 50, 50)) {
          bullets = bullets.filter((b) => b.id !== bullet.id);
          enemies = enemies.filter((e) => e.id !== enemy.id);
          score += 100;

          playSound(enemyDeathAudio);
          createEffect(enemy.x + 25, enemy.y + 25, "explosion");
        }
      });
    });

    enemyBullets.forEach((bullet) => {
      if (isColliding(bullet, { x: mouseX, y: mouseY }, 8, 16, 40, 40)) {
        enemyBullets = enemyBullets.filter((b) => b.id !== bullet.id);
        loseLife();
      }
    });

    enemies.forEach((enemy) => {
      if (isColliding(enemy, { x: mouseX, y: mouseY }, 50, 50, 40, 40)) {
        enemies = enemies.filter((e) => e.id !== enemy.id);
        playSound(enemyDeathAudio);
        createEffect(enemy.x + 25, enemy.y + 25, "explosion");
        loseLife();
      }
    });

    obstacles.forEach((obstacle) => {
      if (
        isColliding(
          obstacle,
          { x: mouseX, y: mouseY },
          obstacle.w,
          obstacle.h,
          40,
          40,
        )
      ) {
        loseLife();

        obstacles = obstacles.map((o) =>
          o.id === obstacle.id
            ? {
                ...o,
                y: -300,
                x: Math.random() * (getGameWidth() - 80) + 40,
                rotation: Math.random() * 360,
              }
            : o,
        );
      }
    });

    powerups.forEach((powerup) => {
      if (isColliding(powerup, { x: mouseX, y: mouseY }, 32, 32, 40, 40)) {
        inventory = [...inventory, powerup];
        powerups = powerups.filter((p) => p.id !== powerup.id);
        createEffect(powerup.x + 15, powerup.y + 15, "pickup");
      }
    });
  }

  function isColliding(a, b, aw, ah, bw, bh) {
    return a.x < b.x + bw && a.x + aw > b.x && a.y < b.y + bh && a.y + ah > b.y;
  }

  function createEffect(x, y, type) {
    effects = [
      ...effects,
      {
        id: effectId++,
        x,
        y,
        type,
        life: type === "muzzle" ? 8 : 18,
      },
    ];
  }

  function loseLife() {
    lives -= 1;
    playSound(damageAudio);
    createEffect(mouseX, mouseY, "damage");

    if (lives <= 0) {
      lives = 0;
      endGame();
    }
  }

  function handleKeyPress(event) {
    if (!gameStarted || gameOver) return;

    const key = event.key.toLowerCase();

    if (key === "q") {
      const rapidFire = inventory.find((item) => item.type === "Rapid Fire");
      usePowerup(rapidFire);
    }

    if (key === "e") {
      const lifePowerup = inventory.find((item) => item.type === "Life");
      usePowerup(lifePowerup);
    }
  }

  function usePowerup(powerup) {
    if (!powerup) return;

    if (powerup.type === "Rapid Fire") {
      rapidFireActive = true;
      fireRate = 150;
      createEffect(mouseX, mouseY, "power");

      setTimeout(() => {
        fireRate = 450;
        rapidFireActive = false;
      }, 5000);
    }

    if (powerup.type === "Life") {
      if (lives < 3) {
        lives += 1;
        createEffect(mouseX, mouseY, "heal");
      }
    }

    inventory = inventory.filter((item) => item.id !== powerup.id);
  }

  function endGame() {
    gameOver = true;

    playSound(gameOverAudio);

    if (musicAudio) {
      musicAudio.pause();
    }

    if (score > highscore) {
      highscore = score;
      localStorage.setItem("spaceHighscore", String(highscore));
    }
  }

  function retry() {
    lives = 3;
    score = 0;
    gameOver = false;

    fireRate = 450;
    lastShot = 0;
    rapidFireActive = false;

    difficulty = 1;
    maxEnemies = 4;
    nextPowerupType = "Rapid Fire";

    bullets = [];
    enemyBullets = [];
    enemies = [];
    powerups = [];
    inventory = [];
    effects = [];

    mouseX = getGameWidth() / 2;
    mouseY = getGameHeight() - 120;

    createObstacles();

    if (musicAudio) {
      musicAudio.currentTime = 0;
      musicAudio.play().catch(() => {});
    }
  }
</script>

<main>
  {#if !gameStarted}
    <section class="start-screen">
      <h1>🚀 Space Cursor Survival</h1>
      <p>Controla a nave com o rato ou com o dedo.</p>
      <p>Destrói inimigos, evita meteoritos e tenta bater o teu highscore.</p>
      <p class="controls">Q = Rapid Fire | E = Vida</p>

      <button on:click={startGame}>PLAY</button>
    </section>
  {:else}
    <h1>🚀 Space Cursor Survival</h1>

    <PlayerHud {lives} {score} {highscore} {fireRate} />
    <Inventory {inventory} />

    <div
      bind:this={gameArea}
      class="game-area"
      on:mousemove={movePlayer}
      on:touchmove|preventDefault={movePlayerTouch}
      on:click={startSound}
      role="application"
      tabindex="0"
    >
      {#if gameOver}
        <GameOver {score} {highscore} on:retry={retry} />
      {/if}

      <div
        class:rapid={rapidFireActive}
        class="player"
        style="left: {mouseX}px; top: {mouseY}px;"
      ></div>

      {#each bullets as bullet}
        <div
          class="bullet"
          style="left: {bullet.x}px; top: {bullet.y}px;"
        ></div>
      {/each}

      {#each enemies as enemy}
        <div class="enemy" style="left: {enemy.x}px; top: {enemy.y}px;">👾</div>
      {/each}

      {#each enemyBullets as bullet}
        <div
          class="enemy-bullet"
          style="left: {bullet.x}px; top: {bullet.y}px;"
        ></div>
      {/each}

      {#each powerups as powerup}
        <div class="powerup" style="left: {powerup.x}px; top: {powerup.y}px;">
          {powerup.type === "Life" ? "❤️" : "⚡"}
        </div>
      {/each}

      {#each obstacles as obstacle}
        <img
          src={meteorImg}
          alt="Meteorito"
          class="obstacle"
          style="
            left: {obstacle.x}px;
            top: {obstacle.y}px;
            width: {obstacle.w}px;
            height: {obstacle.h}px;
            transform: translate(-50%, -50%) rotate({obstacle.rotation ||
            0}deg);
          "
        />
      {/each}

      {#each effects as effect}
        <div
          class="effect {effect.type}"
          style="left: {effect.x}px; top: {effect.y}px;"
        ></div>
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    min-height: 100vh;
    background: #050816;
    color: white;
    font-family: Arial, sans-serif;
    text-align: center;
    padding: 20px;
    overflow-x: hidden;
  }

  h1 {
    margin-bottom: 10px;
  }

  .start-screen {
    min-height: 85vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 14px;
    text-align: center;
  }

  .start-screen h1 {
    font-size: 52px;
    margin-bottom: 10px;
  }

  .start-screen p {
    max-width: 560px;
    color: #cbd5e1;
    font-size: 18px;
    margin: 0;
  }

  .controls {
    color: #facc15 !important;
    font-weight: bold;
  }

  .start-screen button {
    margin-top: 24px;
    background: linear-gradient(90deg, #22c55e, #38bdf8);
    color: white;
    border: none;
    border-radius: 18px;
    padding: 18px 48px;
    font-size: 24px;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 0 0 25px #38bdf8;
  }

  .start-screen button:hover {
    transform: scale(1.08);
  }

  .game-area {
    position: relative;
    width: min(1000px, 95vw);
    height: min(620px, 70vh);
    margin: 20px auto;

    background: radial-gradient(circle at 20% 30%, white 1px, transparent 2px),
      radial-gradient(circle at 80% 20%, white 1px, transparent 2px),
      radial-gradient(circle at 50% 70%, white 1px, transparent 2px),
      radial-gradient(circle at 30% 80%, #93c5fd 1px, transparent 2px),
      radial-gradient(circle at 70% 60%, #c084fc 1px, transparent 2px),
      linear-gradient(180deg, #020617, #0f172a, #111827);

    background-size:
      160px 160px,
      220px 220px,
      300px 300px,
      260px 260px,
      340px 340px,
      cover;

    animation: spaceMove 16s linear infinite;
    border: 3px solid #38bdf8;
    border-radius: 16px;
    overflow: hidden;
    cursor: none;
    touch-action: none;
  }

  @keyframes spaceMove {
    from {
      background-position:
        0 0,
        0 0,
        0 0,
        0 0,
        0 0,
        0 0;
    }

    to {
      background-position:
        0 620px,
        0 460px,
        0 300px,
        0 520px,
        0 380px,
        0 0;
    }
  }

  .player {
    position: absolute;
    transform: translate(-50%, -50%);
    z-index: 5;
    background-image: url("./assets/aviao.png");
    width: 100px;
    height: 100px;
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
  }

  .player.rapid {
    filter: drop-shadow(0 0 18px #facc15);
    animation: rapidPulse 0.25s infinite alternate;
  }

  @keyframes rapidPulse {
    from {
      transform: translate(-50%, -50%) scale(1);
    }

    to {
      transform: translate(-50%, -50%) scale(1.12);
    }
  }

  .bullet {
    position: absolute;
    width: 6px;
    height: 22px;
    background: #22c55e;
    border-radius: 10px;
    box-shadow: 0 0 12px #22c55e;
  }

  .enemy {
    position: absolute;
    font-size: 44px;
    filter: drop-shadow(0 0 10px #ef4444);
  }

  .enemy-bullet {
    position: absolute;
    width: 8px;
    height: 16px;
    background: #ef4444;
    border-radius: 10px;
    box-shadow: 0 0 12px #ef4444;
  }

  .powerup {
    position: absolute;
    font-size: 32px;
    animation: pulse 1s infinite alternate;
    filter: drop-shadow(0 0 12px #facc15);
  }

  .obstacle {
    position: absolute;
    object-fit: contain;
    pointer-events: none;
    z-index: 3;
    filter: drop-shadow(0 0 14px #fb923c);
  }

  .effect {
    position: absolute;
    pointer-events: none;
    transform: translate(-50%, -50%);
    z-index: 10;
  }

  .muzzle {
    width: 18px;
    height: 18px;
    background: #facc15;
    border-radius: 50%;
    box-shadow: 0 0 20px #facc15;
    animation: flash 0.2s ease-out forwards;
  }

  .explosion {
    width: 55px;
    height: 55px;
    background: radial-gradient(
      circle,
      #facc15,
      #f97316,
      #dc2626,
      transparent 70%
    );
    border-radius: 50%;
    animation: explode 0.5s ease-out forwards;
  }

  .damage {
    width: 70px;
    height: 70px;
    border: 4px solid #ef4444;
    border-radius: 50%;
    box-shadow: 0 0 20px #ef4444;
    animation: damageRing 0.45s ease-out forwards;
  }

  .pickup {
    width: 45px;
    height: 45px;
    border: 4px solid #facc15;
    border-radius: 50%;
    box-shadow: 0 0 20px #facc15;
    animation: pickupRing 0.45s ease-out forwards;
  }

  .power {
    width: 85px;
    height: 85px;
    border: 4px solid #facc15;
    border-radius: 50%;
    box-shadow: 0 0 30px #facc15;
    animation: powerRing 0.55s ease-out forwards;
  }

  .heal {
    width: 80px;
    height: 80px;
    border: 4px solid #22c55e;
    border-radius: 50%;
    box-shadow: 0 0 30px #22c55e;
    animation: healRing 0.55s ease-out forwards;
  }

  @keyframes flash {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(1);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(2);
    }
  }

  @keyframes explode {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.4);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(1.8);
    }
  }

  @keyframes damageRing {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.4);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(1.7);
    }
  }

  @keyframes pickupRing {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.3);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(1.5);
    }
  }

  @keyframes powerRing {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.4);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(1.8);
    }
  }

  @keyframes healRing {
    from {
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.4);
    }

    to {
      opacity: 0;
      transform: translate(-50%, -50%) scale(1.8);
    }
  }

  @keyframes pulse {
    from {
      transform: scale(1);
    }

    to {
      transform: scale(1.25);
    }
  }

  @media (max-width: 768px) {
    main {
      padding: 10px;
    }

    h1 {
      font-size: 22px;
    }

    .start-screen h1 {
      font-size: 34px;
    }

    .start-screen p {
      font-size: 15px;
      padding: 0 16px;
    }

    .start-screen button {
      font-size: 20px;
      padding: 15px 38px;
    }

    .game-area {
      width: 95vw;
      height: 65vh;
      border-radius: 12px;
    }

    .player {
      font-size: 32px;
    }

    .enemy {
      font-size: 36px;
    }

    .bullet {
      width: 5px;
      height: 18px;
    }

    .enemy-bullet {
      width: 7px;
      height: 14px;
    }

    .powerup {
      font-size: 28px;
    }
  }

  @media (max-width: 480px) {
    .game-area {
      height: 60vh;
    }

    .player {
      font-size: 30px;
    }

    .enemy {
      font-size: 34px;
    }
  }
</style>