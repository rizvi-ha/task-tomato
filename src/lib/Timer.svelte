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
  
    onMount(() => {
      checkEvents();
      interval = setInterval(checkEvents, 1000); 
    });
  
    onDestroy(() => {
      clearInterval(interval);
    });

    // Reactive statement to calculate hours, minutes, and seconds
    $: hours = Math.floor(status.secondsLeft / 3600);
    $: minutes = Math.floor((status.secondsLeft % 3600) / 60);
    $: seconds = status.secondsLeft % 60;

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
          message = 'Add tasks to begin';
      }
    } 
    


  </script>
  
  <main>
    <div class="timer-box">
      <div class="message">{message}</div>
      <div class="countdown">{formattedTime}</div>
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
    }
    .message {
      font-size: 30px;
    }
    
    .countdown {
      padding : 40px;
      font-size : 60px;
    }
  </style>
  