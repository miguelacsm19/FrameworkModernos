<script>
  import { createEventDispatcher } from 'svelte';
  export let enemies;

  const dispatch = createEventDispatcher();

  function attack(enemy) {
    dispatch('attack', enemy);
  }
</script>

<h2>Inimigos</h2>

{#if enemies.length === 0}
  <p>Todos derrotados! 🎉</p>
{/if}

{#each enemies as enemy}
  <div class="enemy">
    <p>{enemy.name} - HP: {enemy.hp}</p>
    <button on:click={() => attack(enemy)}>Atacar</button>
  </div>
{/each}

<style>
  .enemy {
    margin-bottom: 10px;
  }
</style>