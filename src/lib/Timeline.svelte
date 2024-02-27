<script>

  import { createEventDispatcher } from 'svelte';
  export let length;
  export let events;

  const dispatch = createEventDispatcher();

  function relativeToAbs(start) {
    const result = new Date();
    const roundedHour = result.getHours();
    result.setHours(roundedHour);
    result.setMinutes(0);
    result.setSeconds(0);
    result.setMilliseconds(0);
    result.setTime(result.getTime() + start*60000);
    return result
  }

  function generateTimeline(length) {
    const now = new Date();
    const roundedHour = now.getHours();
    now.setHours(roundedHour);
    now.setMinutes(0);
    now.setSeconds(0);
    now.setMilliseconds(0);

    let ticks = [];
    for (let i = 0; i < length * 12; i++) { 
      let nextTick = new Date(now.getTime() + i * 5 * 60 * 1000); 
      let isHourMark = nextTick.getMinutes() === 0;
      let isHalfHourMark = nextTick.getMinutes() === 30;
      let hours = nextTick.getHours();
      let ampm = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12;
      hours = hours ? hours : 12;
      let minutes = isHourMark ? '00' : nextTick.getMinutes().toString().padStart(2, '0');
      let formattedTime = `${hours}:${minutes} ${ampm}`;
      ticks.push({ time: formattedTime, isHourMark: isHourMark, isHalfHourMark: isHalfHourMark, dateTime: nextTick });
    }

    for (let i = 0; i < events.length; i++) {
      events[i].start = Math.round((events[i].abs_start.getTime() / 60000) - (now.getTime() / 60000));
    }

    return ticks;
  }

  $: ticks = generateTimeline(length);

  function calculateEventStyle(event, length) {
    const totalMinutes = length * 60; // Total timeline length in minutes
    const leftPercentage = (event.start / totalMinutes) * 100;
    const widthPercentage = (event.length / totalMinutes) * 100;
    return `background-color: ${event.color}; 
            left: ${leftPercentage}%; 
            width: ${widthPercentage}%; 
            position: absolute; 
            top: 20px; 
            height: 50%; 
            color: white;
            display: flex; 
            align-items: center; 
            justify-content: center;
            border-radius: 10px;
            user-select: none;`
  }

  $: eventStyles = events.map(event => calculateEventStyle(event, length));

  let accumulatedMovement = 0;

  function updateEventPosition(e, event, index) {

    if (e.detail.mode === "move") {
      const timelineWidth = timelineContainer.offsetWidth;
      const totalMinutes = length * 60;
      const pixelsPerMinute = timelineWidth / totalMinutes;

      accumulatedMovement += e.detail.x; // Accumulate movement

      const movementInMinutes = Math.round(accumulatedMovement / pixelsPerMinute / 5) * 5;

      if (Math.abs(movementInMinutes) >= 5) { // Check if movement exceeds the grid snap unit
        let newStart = event.start + movementInMinutes;
        newStart = newStart - (newStart % 5);
        newStart = Math.max(0, Math.min(newStart, totalMinutes - event.length));

        event.start = newStart;
        events[index] = { ...event };
        events = [...events]; // Trigger reactivity
        dispatch('update', { events });

        accumulatedMovement = 0; // Reset accumulated movement after updating position
      }
    }

  }

  function updateEventLength(e, event, index) {
    if (e.detail.mode === "resize") {
      const timelineWidth = timelineContainer.offsetWidth;
      const totalMinutes = length * 60;
      const pixelsPerMinute = timelineWidth / totalMinutes;

      accumulatedMovement += e.detail.x;
      
      // Calculate resize amount in minutes
      const resizeAmount = Math.round(accumulatedMovement / pixelsPerMinute);

      if (Math.abs(resizeAmount) >= 5){
        let newLength = event.length + resizeAmount;
        newLength = Math.max(5, newLength);
        newLength = newLength - (newLength % 5)

        event.length = newLength;
        events[index] = { ...event };
        events = [...events]; // Trigger reactivity
        dispatch('update', { events });

        accumulatedMovement = 0;
      } 
    }
  }

  let timelineContainer;

  function draggable(node) {
    let x;
    let y;
    let mode; // "move" or "resize"

    function handleMousedown(event) {
      x = event.clientX;
      y = event.clientY;
      const rect = node.getBoundingClientRect();
      mode = (x > rect.right - 11) ? "resize" : "move"; // Threshold for resizing
      
      node.dispatchEvent(new CustomEvent('dragstart', {
        detail: { x, y, mode }
      }));

      window.addEventListener('mousemove', handleMousemove);
      window.addEventListener('mouseup', handleMouseup);
    }

    function handleMousemove(event) {
      const dx = event.clientX - x;
      const dy = event.clientY - y;
      x = event.clientX;
      y = event.clientY;

      node.dispatchEvent(new CustomEvent('dragmove', {
        detail: { x: dx, y: dy, mode }
      }));
    }

    function handleMouseup(event) {
      x = event.clientX;
      y = event.clientY;

      node.dispatchEvent(new CustomEvent('dragend', {
        detail: { x, y }
      }));

      window.removeEventListener('mousemove', handleMousemove);
      window.removeEventListener('mouseup', handleMouseup);
    }

    node.addEventListener('mousedown', handleMousedown);

    return {
      destroy() {
        node.removeEventListener('mousedown', handleMousedown);
      }
    };
  }

  function handleDragEnd() {
    accumulatedMovement = 0; // Reset on drag end
  }

  let playheadElement;

  // Function to calculate playhead's left position as a percentage
  function updatePlayheadPosition(length) {
    const now = new Date();

    // To get the timeline to scroll if time passes onto the next hour
    if (now.getMinutes() == 0) {
      ticks = generateTimeline(length);
    }

    const start = ticks[0].dateTime;
    const totalDuration = length*60;
    const currentTimeOffset = now.getMinutes() - start.getMinutes();
    const leftPercentage = (currentTimeOffset / totalDuration) * 100;
    return leftPercentage;
  }

  // Reactive statement to update playhead position
  $: if (playheadElement) {
    const leftPos = updatePlayheadPosition(length);
    playheadElement.style.left = `${leftPos}%`;
  }

  // Update playhead position every 10 seconds
  setInterval(() => {
    if (playheadElement) {
      const leftPos = updatePlayheadPosition(length);
      playheadElement.style.left = `${leftPos}%`;
    }
  }, 10000);

  $: for (let i = 0; i < events.length; i++) {
      events[i].abs_start = relativeToAbs(events[i].start);
  }

  function handleDrop(event) {
    event.preventDefault();
    const data = event.dataTransfer.getData("application/task");
    const task = JSON.parse(data);

    const newEvent = {
      id: Date.now(), 
      start: 0, // You might want to calculate this based on drop position
      length: 25, 
      color: task.color, 
      label: task.title,
      abs_start: relativeToAbs(0) 
    };

    events = [...events, newEvent];
    dispatch('update', { events }); 
  }

  function handleRightClick(event, eventId) {
    event.preventDefault(); // Prevent the default context menu
    events = events.filter(e => e.id !== eventId); // Remove the event from the array
    dispatch('update', { events }); // Dispatch an update to ensure reactivity
  }

  function handleDragOver(event) {
    event.preventDefault();
  }

</script>
  
<div role="presentation" class="timeline-container" on:drop={handleDrop} on:dragover={handleDragOver} bind:this={timelineContainer}>

  {#each ticks as { time, isHourMark, isHalfHourMark }, index}

    <div class="tick" style="position: relative;">
      <div class={isHourMark ? 'hour-mark' : (isHalfHourMark ? 'half-hour-mark' : 'minute-mark')}></div>
      {#if isHourMark}
        <div class="hour-label">&nbsp;&nbsp;{time}</div>
      {/if}
    </div>

  {/each}

  {#each events as event, index (event.id)}

    <div use:draggable
         style="{eventStyles[index]}"
         class="event"
         on:dragmove="{(e) => {
          if (e.detail.mode === 'move') {
            updateEventPosition(e, event, index);
          } else if (e.detail.mode === 'resize') {
            updateEventLength(e, event, index);
          }}}"

         on:dragend="{handleDragEnd}"
         on:contextmenu="{(e) => handleRightClick(e, event.id)}"
         role="gridcell"
         tabindex="0"
         aria-label="Draggable event">

      {event.label}

    </div>

  {/each}

  <div class="playhead" bind:this={playheadElement}></div>

</div>

  
<style>

  .timeline-container {
    display: flex;
    width: 100%;
    height: 100px;
    white-space: nowrap;
    border-top: 2px solid #ff574b;
    position: relative;
    overflow: hidden;
  }

  .tick {
    flex: 1;
    position: relative;
    user-select: none;
  }

  .hour-mark, .minute-mark, .half-hour-mark {
    position: absolute;
    width: 100%;
    left: 0;
    user-select: none;
  }

  .hour-mark {
    border-left: 3px solid #787878;
    height: 100px; 
    top: 0;
  }

  .minute-mark {
    border-left: 1px solid #787878;
    height: 10px; 
    top: 0;
  }

  .half-hour-mark {
    border-left: 1px solid #787878;
    height: 17px; 
    top: 0;
  }

  .hour-label {
    position: absolute;
    top: 0;
    background: #ffffff00; 
    color: #e7b07d;
    padding-top: 80px;
    padding-right: 10px;
    user-select: none; 
  }

  .event {
  position: relative;
  cursor: grab; 
  overflow: hidden;
}

.event::after {
  content: '';
  position: absolute;
  top: 0;
  right: 0;
  width: 10px; /* Width of the resize area */
  height: 100%;
  background-color: rgba(161, 30, 0, 0.5); /* Semi-transparent white for indication */
  cursor: col-resize; /* Cursor indicates resize action */
  border-radius: 10px;
}

.playhead {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 2px; 
  background-color: #136800; 
  z-index: 10; 
}

</style>