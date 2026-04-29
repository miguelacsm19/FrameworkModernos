<script>
  export let inventory = [];

  $: rapidFireCount = inventory.filter(item => item.type === 'Rapid Fire').length;
  $: lifeCount = inventory.filter(item => item.type === 'Life').length;
</script>

<div class="inventory">
  <h2>🎒 Inventário</h2>

  <div class="items">
    <div class="item">
      ⚡ Rapid Fire: {rapidFireCount}
      <span>Tecla Q</span>
    </div>

    <div class="item">
      ❤️ Vida: {lifeCount}
      <span>Tecla E</span>
    </div>
  </div>
</div>

<style>
  .inventory {
    margin-top: 15px;
  }

  .items {
    display: flex;
    justify-content: center;
    gap: 15px;
  }

  .item {
    background: #111827;
    border: 2px solid #38bdf8;
    border-radius: 12px;
    padding: 10px 16px;
    font-weight: bold;
  }

  span {
    display: block;
    margin-top: 5px;
    color: #facc15;
    font-size: 14px;
  }
</style>