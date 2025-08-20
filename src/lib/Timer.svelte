<script>
    import { onMount, onDestroy } from 'svelte';
    import App from '../App.svelte';
    export let events;
  

    let status = {
      secondsLeft : 0,
      state : 'no_timer', // 'no_timer', 'task_timer', 'break_timer'
      doubleBooked : false,
      eventName : '',
    };

    let interval;
    let message;
    let startSound;
    let endSound;
    let quote = { content: 'Loading quote...', author: '' };
  
    const checkEvents = () => {

      const now = new Date();
      const ongoingEvents = events.filter(event => {
        const eventEnd = new Date(event.abs_start.getTime() + (event.length * 60000));
        return now >= event.abs_start && now <= eventEnd;
      });
  
      if (ongoingEvents.length > 1) {

        status.doubleBooked = true;
        status.state = 'no_timer';
        status.secondsLeft = 0;
        status.eventName = '';

      } else if (ongoingEvents.length === 1) {

        const event = ongoingEvents[0];
        const eventEnd = event.abs_start.getTime() + (event.length * 60000);

        status.secondsLeft = Math.round((eventEnd - now.getTime()) / 1000); 
        status.state = 'task_timer';
        status.doubleBooked = false;
        status.eventName = event.label;

      } else {

        // Filter all upcoming events
         const upcomingEvents = events.filter(event => now < event.abs_start);
        // Sort them by their start time to find the earliest
        upcomingEvents.sort((a, b) => a.abs_start - b.abs_start);

        if (upcomingEvents.length > 0) {

            const nextEvent = upcomingEvents[0]; 
            status.secondsLeft = Math.round((nextEvent.abs_start.getTime() - now.getTime()) / 1000); 
            status.state = 'break_timer';
            status.doubleBooked = false;
            status.eventName = '';

        } else {
            status.secondsLeft = 0;
            status.state = 'no_timer';
            status.doubleBooked = false;
            status.eventName = '';
        }
      }
    };

    const fetchRandomQuote = async () => {
    
      const response = await fetch(`https://zenquotes.io/api/random`);
    
      if (response.ok) {
        const data = await response.json();
        // Assuming the API returns an array of quotes, pick the first one
        if (data.length > 0) {
          quote = { content: data[0].q, author: data[0].a };
        } else {
          quote = { content: 'No quotes available', author: '' };
        }
      } else {
        quote = { content: 'Failed to load quote', author: '' };
      }
    };
      
    onMount(() => {
      checkEvents();
      fetchRandomQuote();
      interval = setInterval(checkEvents, 1000);
      
      startSound = new Audio('/break-end.mp3');
      endSound = new Audio('/task-end.mp3')
    });
  
    onDestroy(() => {
      clearInterval(interval);
    });

    // Reactive statement to calculate hours, minutes, and seconds
    $: hours = Math.floor(status.secondsLeft / 3600);
    $: minutes = Math.floor((status.secondsLeft % 3600) / 60);
    $: seconds = status.secondsLeft % 60;

    $: if (status.secondsLeft === 1 && status.state !== 'break_timer') {
        startSound.play().catch(error => console.error("Error playing sound:", error));
      }

    $: if (status.secondsLeft === 1 && status.state !== 'task_timer') {
        endSound.play().catch(error => console.error("Error playing sound:", error));
      }

    // Format the time nicely
    $: formattedTime = hours > 0
        ? `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
        : `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;


    $: {
      if (status.state == 'task_timer') {
        message = `You are currently working on: ${status.eventName}`;
      }
      else if (status.state == 'break_timer') {
        message = 'Enjoy your break! Check back in:';
      }
      else
      {
        if (status.doubleBooked)
          message = 'Only one task at a time!';
        else 
          message = 'Drag tasks onto the timeline to begin';
      }
    } 

    $: document.title = status.state === 'no_timer' ?  'task-tomato' : (`${formattedTime} - ${status.state === 'task_timer' ? status.eventName : 'Break Time!'}`);
    


  </script>
  
  <main>
    <div class="timer-box">
      <div class="message"><b>{message}</b></div>
      <div class="countdown">{formattedTime}</div>
      <div class="quote"><br><br><i>"{quote.content}" - {quote.author}</i></div>
    </div>
  </main>

  <style>
    .timer-box {
      background-color: #2b2b2b;
      color: white;
      padding: 20px;
      text-align: center;
      border-radius: 8px;
      height: 400px;
      line-height: 1;
      overflow : auto;
    }
    .message {
      font-size: 30px;
      line-height: 3;
    }
    
    .countdown {
      padding : 40px;
      font-size : 100px;
    }

    .quote {
      font-size : 20px;
    }
  </style>
  
