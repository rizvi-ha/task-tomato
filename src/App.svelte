<script>

  import { onMount } from 'svelte';
  import Timeline from './lib/Timeline.svelte';
  import Todo from './lib/Todo.svelte';

  let length = 6;

  function incrementLength() {
    if (length < 10)
      length += 1;
  }

  function decrementLength() {
    if (length > 4) 
      length -= 1;  
  }

  onMount(() => {
    // initialization if needed
  });

</script>

<main>
  <div class="timeline-cont">
    <Timeline {length} />
    <div class="length-controls">
      <button on:click={incrementLength}>+</button>
      <button on:click={decrementLength}>-</button>
    </div>
  </div>
  <div class="todo-container">
    <Todo />
  </div>
</main>

<style>

  main {
    display: flex;
    justify-content: flex-start;
    gap: 20px;
    padding: 0px;
    width: 100%;
    flex-direction: column;
    height: 100vh;
  }

  .timeline-cont {
    padding: 10px;
    width: 95vw; 
    position: relative;
  }

  .length-controls {
    position: absolute;
    right: 0px;
    top: 50px;
    display: flex;
    flex-direction: column;
  }

  .length-controls button {
    font-size: 0.8rem;
    margin-top: 5px;
    padding: 5px 10px;
    width: auto;
    height: auto;
  }

  .todo-container {
    padding: 10px;
    width: 25vw;
  }

</style>
