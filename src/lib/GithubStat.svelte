<script>
    export let label = "";
    export let value = "";
</script>

<div class="item">
  <dt>{label}</dt>
  <dd>{value}</dd>
</div>

<style>
  .item {
    display: flex;
    flex-direction: column;
    padding: 0.5rem;
    min-width: 50px; /* Adjust based on your content */
  }

  dt {
    /* font-weight: bold; */
    font-size: 1.3rem;
    color: color-mix(in srgb, canvastext, canvas 20%);
  }

  dd {
    margin: 0;
    font-size: 1.4rem;
  }
</style>