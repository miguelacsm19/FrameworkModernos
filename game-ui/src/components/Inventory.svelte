<script>
  import { createEventDispatcher } from 'svelte';

  export let inventory = [];

  const dispatch = createEventDispatcher();

  function usePowerup(item) {
    dispatch('usePowerup', item);
  }
</script>

<div class="inventory">
  <h2>🎒 Inventário</h2>

  {#if inventory.length === 0}
    <p>Sem powerups</p>
  {:else}
    {#each inventory as item}
      <button on:click={() => usePowerup(item)}>
        ⚡ Usar {item.type}
      </button>
    {/each}
  {/if}
</div>

<style>
  .inventory {
    margin-top: 15px;
  }

  button {
    background: #facc15;
    color: #111827;
    border: none;
    border-radius: 10px;
    padding: 10px 14px;
    margin: 5px;
    font-weight: bold;
    cursor: pointer;
  }

  button:hover {
    background: #fde047;
  }

  p {
    color: #cbd5e1;
  }
</style>