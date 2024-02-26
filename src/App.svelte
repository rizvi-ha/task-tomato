<script>

  import { onMount } from 'svelte';
  import Timeline from './lib/Timeline.svelte';
  import Todo from './lib/Todo.svelte';
  import Timer from './lib/Timer.svelte';

  let length = 6;

  let events = JSON.parse(localStorage.getItem('events')) || [];

  if (events.length !== 0)
  {
    events = events.map(event => ({
    ...event,
    // Convert abs_start back to a Date object
    abs_start: new Date(event.abs_start)
    }));
  }
    
  function incrementLength() {
    if (length < 10)
      length += 1;
      localStorage.setItem('length', length.toString());
  }

  function decrementLength() {
    if (length > 4) 
      length -= 1;  
      localStorage.setItem('length', length.toString());
  }

  onMount(() => {
    const storedLength = localStorage.getItem('length');
    if (storedLength !== null) {
      length = parseInt(storedLength, 10);
    }
  });

  function saveEvents() {
    localStorage.setItem('events', JSON.stringify(events));
  }

  function handleEventsUpdate(e) {
    events = e.detail.events;
    saveEvents();
  }

</script>

<main>
  <header>
    <img src="/icon.png" alt="Logo" class="logo"> <!-- Add your logo path -->
    <span class="site-name">task-tomato</span>
  </header>
  <div class="timeline-cont">
    <Timeline {length} {events} on:update="{e => handleEventsUpdate(e)}" />
    <div class="length-controls">
      <button on:click={incrementLength}>+</button>
      <button on:click={decrementLength}>-</button>
    </div>
  </div>
  <div class = "foot-area">
    <div class="todo-container">
      <Todo {events} on:update="{e => handleEventsUpdate(e)}"/>
    </div>
    <div class="timer-container">
      <Timer {events}/>
    </div>
  </div>
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
    gap: 0px;
    padding: 0px;
    width: 100%;
    flex-direction: column;
    height: 100vh;
  }

  .timeline-cont {
    padding: 2vw;
    width: 95vw; 
    position: relative;
  }

  .length-controls {
    position: absolute;
    right: 40px;
    top: 70px;
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

  .foot-area {
    display: flex;
    justify-content: flex-start;
    gap: 20px;
    padding: 0px;
    width: 100vw;
    flex-direction: row;
  }

  .timer-container{
    width : 70vw;
    height : 400px;
  }

</style>
