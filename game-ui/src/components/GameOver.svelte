<script>
  import { createEventDispatcher } from 'svelte';

  export let score;
  export let highscore;

  const dispatch = createEventDispatcher();

  function retry() {
    dispatch('retry');
  }
</script>

<div class="overlay">
  <div class="box">
    <h2>💀 Game Over</h2>
    <p>Score final: {score}</p>
    <p>Highscore: {highscore}</p>

    <button on:click={retry}>Retry</button>
  </div>
</div>

<style>
  .overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.82);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 20;
  }

  .box {
    background: #111827;
    border: 3px solid #ef4444;
    border-radius: 18px;
    padding: 35px;
    width: 320px;
    text-align: center;
  }

  h2 {
    font-size: 32px;
    margin-bottom: 10px;
  }

  p {
    font-size: 20px;
  }

  button {
    background: #22c55e;
    color: white;
    border: none;
    border-radius: 12px;
    padding: 12px 24px;
    font-size: 18px;
    cursor: pointer;
    font-weight: bold;
  }

  button:hover {
    background: #16a34a;
  }
</style>