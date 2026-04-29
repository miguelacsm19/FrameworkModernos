<script>
  import { onMount } from 'svelte';
  import PlayerHud from './components/PlayerHud.svelte';
  import Inventory from './components/Inventory.svelte';
  import GameOver from './components/GameOver.svelte';

  let gameArea;
  let mouseX = 400;
  let mouseY = 300;

  let lives = 3;
  let score = 0;
  let highscore = 0;
  let gameOver = false;

  let fireRate = 450;
  let lastShot = 0;

  let bullets = [];
  let enemyBullets = [];
  let enemies = [];
  let powerups = [];
  let obstacles = [];
  let inventory = [];

  let enemyId = 1;
  let bulletId = 1;
  let powerupId = 1;

  onMount(() => {
    highscore = Number(localStorage.getItem('spaceHighscore')) || 0;
    createObstacles();

    const loop = setInterval(updateGame, 30);
    const spawnEnemies = setInterval(spawnEnemy, 1300);
    const spawnPowerups = setInterval(spawnPowerup, 7000);
    const enemyShoot = setInterval(enemyFire, 1200);

    return () => {
      clearInterval(loop);
      clearInterval(spawnEnemies);
      clearInterval(spawnPowerups);
      clearInterval(enemyShoot);
    };
  });

  function createObstacles() {
    obstacles = [
      { id: 1, x: 250, y: 180, w: 80, h: 80 },
      { id: 2, x: 580, y: 390, w: 100, h: 70 },
      { id: 3, x: 760, y: 160, w: 70, h: 110 }
    ];
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
  }

  function spawnEnemy() {
    if (gameOver) return;

    enemies = [
      ...enemies,
      {
        id: enemyId++,
        x: Math.random() * 850 + 50,
        y: -40,
        hp: 2,
        speed: Math.random() * 2 + 1.5
      }
    ];
  }

  function spawnPowerup() {
    if (gameOver) return;

    powerups = [
      ...powerups,
      {
        id: powerupId++,
        x: Math.random() * 850 + 50,
        y: Math.random() * 400 + 80,
        type: 'Rapid Fire'
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
          x: enemy.x + 15,
          y: enemy.y + 35,
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

    checkCollisions();
  }

  function checkCollisions() {
    bullets.forEach(bullet => {
      enemies.forEach(enemy => {
        if (isColliding(bullet, enemy, 8, 8, 35, 35)) {
          enemy.hp -= 1;
          bullets = bullets.filter(b => b.id !== bullet.id);

          if (enemy.hp <= 0) {
            enemies = enemies.filter(e => e.id !== enemy.id);
            score += 100;
          }
        }
      });
    });

    enemyBullets.forEach(bullet => {
      if (isColliding(bullet, { x: mouseX, y: mouseY }, 8, 8, 40, 40)) {
        enemyBullets = enemyBullets.filter(b => b.id !== bullet.id);
        loseLife();
      }
    });

    enemies.forEach(enemy => {
      if (isColliding(enemy, { x: mouseX, y: mouseY }, 35, 35, 40, 40)) {
        enemies = enemies.filter(e => e.id !== enemy.id);
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
      if (isColliding(powerup, { x: mouseX, y: mouseY }, 28, 28, 40, 40)) {
        inventory = [...inventory, powerup];
        powerups = powerups.filter(p => p.id !== powerup.id);
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

  function loseLife() {
    lives -= 1;

    if (lives <= 0) {
      lives = 0;
      endGame();
    }
  }

  function usePowerup(powerup) {
    if (powerup.type === 'Rapid Fire') {
      fireRate = 150;

      setTimeout(() => {
        fireRate = 450;
      }, 6000);
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

    bullets = [];
    enemyBullets = [];
    enemies = [];
    powerups = [];
    inventory = [];

    mouseX = 400;
    mouseY = 500;
    createObstacles();
  }
</script>

<main>
  <h1>🚀 Space Cursor Survival</h1>

  <PlayerHud {lives} {score} {highscore} {fireRate} />
  <Inventory {inventory} on:usePowerup={(e) => usePowerup(e.detail)} />

  <div
    bind:this={gameArea}
    class="game-area"
    on:mousemove={movePlayer}
    on:click={shoot}
  >
    {#if gameOver}
      <GameOver {score} {highscore} on:retry={retry} />
    {/if}

    <div class="player" style="left: {mouseX}px; top: {mouseY}px;">🚀</div>

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
      <div class="powerup" style="left: {powerup.x}px; top: {powerup.y}px;">⚡</div>
    {/each}

    {#each obstacles as obstacle}
      <div
        class="obstacle"
        style="left: {obstacle.x}px; top: {obstacle.y}px; width: {obstacle.w}px; height: {obstacle.h}px;"
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
    background: radial-gradient(circle, #172554, #020617);
    border: 3px solid #38bdf8;
    border-radius: 16px;
    overflow: hidden;
    cursor: none;
  }

  .player {
    position: absolute;
    transform: translate(-50%, -50%);
    font-size: 38px;
    z-index: 5;
  }

  .bullet {
    position: absolute;
    width: 6px;
    height: 18px;
    background: #22c55e;
    border-radius: 10px;
  }

  .enemy {
    position: absolute;
    font-size: 32px;
  }

  .enemy-bullet {
    position: absolute;
    width: 8px;
    height: 16px;
    background: #ef4444;
    border-radius: 10px;
  }

  .powerup {
    position: absolute;
    font-size: 28px;
    animation: pulse 1s infinite alternate;
  }

  .obstacle {
    position: absolute;
    background: #64748b;
    border: 2px solid #94a3b8;
    border-radius: 12px;
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