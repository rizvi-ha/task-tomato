<script>

  import { onMount } from 'svelte';
  import Timeline from './lib/Timeline.svelte';
  import Todo from './lib/Todo.svelte';

  let length = 6;

  const test_time = new Date();
  test_time.setHours(23);
  test_time.setMinutes(30);

  let events = [
  {
    "id": 1,
    "start": 0,
    "length": 100,
    "color": "#ea4b07",
    "label": "Study",
    "abs_start" : test_time
  }, 
  {
    "id": 2,
    "start": 0,
    "length": 40,
    "color": "#ea0000",
    "label": "Eat",
    "abs_start" : test_time
  }];

  function incrementLength() {
    if (length < 10)
      length += 1;
  }

  function decrementLength() {
    if (length > 4) 
      length -= 1;  
  }

  onMount(() => {
    //
  });

</script>

<main>
  <header>
    <img src="/icon.png" alt="Logo" class="logo"> <!-- Add your logo path -->
    <span class="site-name">task-tomato</span>
  </header>
  <div class="timeline-cont">
    <Timeline {length} {events} on:update="{e => events = e.detail.events}" />
    <div class="length-controls">
      <button on:click={incrementLength}>+</button>
      <button on:click={decrementLength}>-</button>
    </div>
  </div>
  <div class="todo-container">
    <Todo />
  </div>
  <pre>{JSON.stringify(events, null, 2)}</pre>

</main>



<style>

  header {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
  }

  .logo {
    width: 80px; /* Adjust based on your logo's size */
    height: auto;
    margin-right: 10px; /* Gives some space between the logo and the site name */
  }

  .site-name {
    font-size: 40px; /* Adjust size as needed */
    font-weight: 700; /* Optional: if you want the font to be bold */
  }

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
