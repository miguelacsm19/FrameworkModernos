<script>
  export let life;
</script>

<h2>Vida</h2>

<div class="bar">
  <div class="fill" style="width: {life}%"></div>
</div>

<p>{life} HP</p>

<style>
  .bar {
    width: 200px;
    height: 20px;
    background: #ccc;
  }

  .fill {
    height: 100%;
    background: green;
  }
</style>