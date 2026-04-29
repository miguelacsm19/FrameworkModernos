<script>
  // @ts-nocheck

  import { onMount } from 'svelte';
  import PlayerHud from './components/PlayerHud.svelte';
  import Inventory from './components/Inventory.svelte';
  import GameOver from './components/GameOver.svelte';

  let gameArea;
  let mouseX = 400;
  let mouseY = 500;

  let lives = 3;
  let score = 0;
  let highscore = 0;
  let gameOver = false;

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

  onMount(() => {
    highscore = Number(localStorage.getItem('spaceHighscore')) || 0;
    createObstacles();

    const loop = setInterval(updateGame, 30);
    const spawnEnemies = setInterval(spawnEnemy, 1500);
    const spawnPowerups = setInterval(spawnPowerup, 10000);
    const enemyShoot = setInterval(enemyFire, 1500);
    const increaseDifficulty = setInterval(makeHarder, 12000);
    const autoShoot = setInterval(shoot, 50);

    window.addEventListener('keydown', handleKeyPress);

    return () => {
      clearInterval(loop);
      clearInterval(spawnEnemies);
      clearInterval(spawnPowerups);
      clearInterval(enemyShoot);
      clearInterval(increaseDifficulty);
      clearInterval(autoShoot);
      window.removeEventListener('keydown', handleKeyPress);
    };
  });

  function createObstacles() {
    obstacles = [
      { id: 1, x: 220, y: -100, w: 55, h: 55, speed: 2.2 },
      { id: 2, x: 620, y: -350, w: 70, h: 70, speed: 1.8 },
      { id: 3, x: 850, y: -600, w: 60, h: 60, speed: 2 }
    ];
  }

  function makeHarder() {
    if (gameOver) return;

    difficulty += 0.2;

    if (maxEnemies < 8) {
      maxEnemies += 1;
    }
  }

  function movePlayer(event) {
    if (gameOver) return;

    const rect = gameArea.getBoundingClientRect();
    mouseX = event.clientX - rect.left;
    mouseY = event.clientY - rect.top;
  }

  function shoot() {
    if (gameOver) return;

    const now = Date.now();

    if (now - lastShot < fireRate) return;

    lastShot = now;

    bullets = [
      ...bullets,
      {
        id: bulletId++,
        x: mouseX + 15,
        y: mouseY,
        speed: 14
      }
    ];

    createEffect(mouseX + 15, mouseY - 20, 'muzzle');
  }

  function spawnEnemy() {
    if (gameOver) return;
    if (enemies.length >= maxEnemies) return;

    enemies = [
      ...enemies,
      {
        id: enemyId++,
        x: Math.random() * 850 + 50,
        y: -60,
        hp: 1,
        speed: Math.random() * 1.5 + difficulty
      }
    ];
  }

  function spawnPowerup() {
    if (gameOver) return;
    if (powerups.length >= 1) return;

    const types = ['Rapid Fire', 'Life'];
    const randomType = types[Math.floor(Math.random() * types.length)];

    powerups = [
      ...powerups,
      {
        id: powerupId++,
        x: Math.random() * 850 + 50,
        y: Math.random() * 400 + 80,
        type: randomType
      }
    ];
  }

  function enemyFire() {
    if (gameOver) return;

    enemies.forEach(enemy => {
      enemyBullets = [
        ...enemyBullets,
        {
          id: bulletId++,
          x: enemy.x + 22,
          y: enemy.y + 45,
          speed: 6
        }
      ];
    });
  }

  function updateGame() {
    if (gameOver) return;

    bullets = bullets
      .map(b => ({ ...b, y: b.y - b.speed }))
      .filter(b => b.y > -20);

    enemyBullets = enemyBullets
      .map(b => ({ ...b, y: b.y + b.speed }))
      .filter(b => b.y < 650);

    enemies = enemies
      .map(e => ({ ...e, y: e.y + e.speed }))
      .filter(e => e.y < 650);

    obstacles = obstacles.map(o => {
      let newY = o.y + o.speed + difficulty * 0.2;

      if (newY > 650) {
        return {
          ...o,
          x: Math.random() * 900 + 30,
          y: -Math.random() * 500 - 100,
          speed: Math.random() * 1.5 + 1.5
        };
      }

      return {
        ...o,
        y: newY
      };
    });

    effects = effects
      .map(effect => ({ ...effect, life: effect.life - 1 }))
      .filter(effect => effect.life > 0);

    checkCollisions();
  }

  function checkCollisions() {
    bullets.forEach(bullet => {
      enemies.forEach(enemy => {
        if (isColliding(bullet, enemy, 8, 8, 50, 50)) {
          bullets = bullets.filter(b => b.id !== bullet.id);
          enemies = enemies.filter(e => e.id !== enemy.id);
          score += 100;

          createEffect(enemy.x + 25, enemy.y + 25, 'explosion');
        }
      });
    });

    enemyBullets.forEach(bullet => {
      if (isColliding(bullet, { x: mouseX, y: mouseY }, 8, 16, 40, 40)) {
        enemyBullets = enemyBullets.filter(b => b.id !== bullet.id);
        loseLife();
      }
    });

    enemies.forEach(enemy => {
      if (isColliding(enemy, { x: mouseX, y: mouseY }, 50, 50, 40, 40)) {
        enemies = enemies.filter(e => e.id !== enemy.id);
        createEffect(enemy.x + 25, enemy.y + 25, 'explosion');
        loseLife();
      }
    });

    obstacles.forEach(obstacle => {
      if (isColliding(obstacle, { x: mouseX, y: mouseY }, obstacle.w, obstacle.h, 40, 40)) {
        loseLife();
        mouseX = 80;
        mouseY = 550;
      }
    });

    powerups.forEach(powerup => {
      if (isColliding(powerup, { x: mouseX, y: mouseY }, 32, 32, 40, 40)) {
        inventory = [...inventory, powerup];
        powerups = powerups.filter(p => p.id !== powerup.id);
        createEffect(powerup.x + 15, powerup.y + 15, 'pickup');
      }
    });
  }

  function isColliding(a, b, aw, ah, bw, bh) {
    return (
      a.x < b.x + bw &&
      a.x + aw > b.x &&
      a.y < b.y + bh &&
      a.y + ah > b.y
    );
  }

  function createEffect(x, y, type) {
    effects = [
      ...effects,
      {
        id: effectId++,
        x,
        y,
        type,
        life: type === 'muzzle' ? 8 : 18
      }
    ];
  }

  function loseLife() {
    lives -= 1;
    createEffect(mouseX, mouseY, 'damage');

    if (lives <= 0) {
      lives = 0;
      endGame();
    }
  }

  function handleKeyPress(event) {
    if (gameOver) return;

    const key = event.key.toLowerCase();

    if (key === 'q') {
      const rapidFire = inventory.find(item => item.type === 'Rapid Fire');
      usePowerup(rapidFire);
    }

    if (key === 'e') {
      const lifePowerup = inventory.find(item => item.type === 'Life');
      usePowerup(lifePowerup);
    }
  }

  function usePowerup(powerup) {
    if (!powerup) return;

    if (powerup.type === 'Rapid Fire') {
      rapidFireActive = true;
      fireRate = 150;
      createEffect(mouseX, mouseY, 'power');

      setTimeout(() => {
        fireRate = 450;
        rapidFireActive = false;
      }, 5000);
    }

    if (powerup.type === 'Life') {
      if (lives < 3) {
        lives += 1;
        createEffect(mouseX, mouseY, 'heal');
      }
    }

    inventory = inventory.filter(item => item.id !== powerup.id);
  }

  function endGame() {
    gameOver = true;

    if (score > highscore) {
      highscore = score;
      localStorage.setItem('spaceHighscore', String(highscore));
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

    bullets = [];
    enemyBullets = [];
    enemies = [];
    powerups = [];
    inventory = [];
    effects = [];

    mouseX = 400;
    mouseY = 500;

    createObstacles();
  }
</script>

<main>
  <h1>🚀 Space Cursor Survival</h1>

  <PlayerHud {lives} {score} {highscore} {fireRate} />
  <Inventory {inventory} />

  <div
    bind:this={gameArea}
    class="game-area"
    on:mousemove={movePlayer}
    role="application"
    tabindex="0"
  >
    {#if gameOver}
      <GameOver {score} {highscore} on:retry={retry} />
    {/if}

    <div class:rapid={rapidFireActive} class="player" style="left: {mouseX}px; top: {mouseY}px;">🚀</div>

    {#each bullets as bullet}
      <div class="bullet" style="left: {bullet.x}px; top: {bullet.y}px;"></div>
    {/each}

    {#each enemies as enemy}
      <div class="enemy" style="left: {enemy.x}px; top: {enemy.y}px;">👾</div>
    {/each}

    {#each enemyBullets as bullet}
      <div class="enemy-bullet" style="left: {bullet.x}px; top: {bullet.y}px;"></div>
    {/each}

    {#each powerups as powerup}
      <div class="powerup" style="left: {powerup.x}px; top: {powerup.y}px;">
        {powerup.type === 'Life' ? '❤️' : '⚡'}
      </div>
    {/each}

    {#each obstacles as obstacle}
      <div
        class="obstacle"
        style="left: {obstacle.x}px; top: {obstacle.y}px; width: {obstacle.w}px; height: {obstacle.h}px;"
      ></div>
    {/each}

    {#each effects as effect}
      <div
        class="effect {effect.type}"
        style="left: {effect.x}px; top: {effect.y}px;"
      ></div>
    {/each}
  </div>
</main>

<style>
  main {
    min-height: 100vh;
    background: #050816;
    color: white;
    font-family: Arial, sans-serif;
    text-align: center;
    padding: 20px;
  }

  h1 {
    margin-bottom: 10px;
  }

  .game-area {
    position: relative;
    width: 1000px;
    height: 620px;
    margin: 20px auto;

    background:
      radial-gradient(circle at 20% 30%, white 1px, transparent 2px),
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
  }

  @keyframes spaceMove {
    from {
      background-position: 0 0, 0 0, 0 0, 0 0, 0 0, 0 0;
    }

    to {
      background-position: 0 620px, 0 460px, 0 300px, 0 520px, 0 380px, 0 0;
    }
  }

  .player {
    position: absolute;
    transform: translate(-50%, -50%);
    font-size: 38px;
    z-index: 5;
    filter: drop-shadow(0 0 10px #38bdf8);
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
    background: linear-gradient(135deg, #f97316, #7c2d12);
    border: 2px solid #fed7aa;
    border-radius: 50%;
    box-shadow: 0 0 18px #fb923c;
  }

  .obstacle::after {
    content: "";
    position: absolute;
    width: 70px;
    height: 18px;
    background: linear-gradient(90deg, rgba(251, 146, 60, 0.8), transparent);
    right: 75%;
    top: 35%;
    transform: rotate(20deg);
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
    background: radial-gradient(circle, #facc15, #f97316, #dc2626, transparent 70%);
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
</style>