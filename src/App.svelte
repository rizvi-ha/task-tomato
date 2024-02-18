<script>
  import Timeline from './lib/Timeline.svelte';
</script>

<main>
  <div class=timeline-container>
    <Timeline length={6} />
  </div>
  
</main>

<style>
  main {
    position: absolute;
    top: 0;
    left: 0;
  }

  .timeline-container {
    position: absolute;
    top: 0;
    left: 0;
    padding: 10px;
  }
</style>
