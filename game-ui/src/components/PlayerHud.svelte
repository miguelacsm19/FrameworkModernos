<script>
  export let lives;
  export let score;
  export let highscore;
  export let fireRate;
</script>

<div class="hud">
  <div>
    <span class="vida"></span>
    Vidas: {lives}/3
  </div>
  <div>⭐ Score: {score}</div>
  <div>🏆 Highscore: {highscore}</div>
  <div>🔫 Fire Rate: {fireRate === 150 ? "Rápido" : "Normal"}</div>
</div>

<style>
  .hud {
    display: flex;
    justify-content: center;
    gap: 25px;
    background: #111827;
    border: 2px solid #38bdf8;
    border-radius: 12px;
    padding: 12px;
    width: fit-content;
    margin: 0 auto;
    font-weight: bold;
  }

  .vida {
    width: 1rem;
    height: 1rem;
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
    background-image: url("./../assets/curacaum.png");
    position: static;
    display: inline-block;
  }
</style>
