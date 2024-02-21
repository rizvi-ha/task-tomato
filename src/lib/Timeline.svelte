<script>

  import { createEventDispatcher } from 'svelte';
  export let length;
  export let events;

  const dispatch = createEventDispatcher();

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
      let hours = nextTick.getHours();
      let ampm = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12;
      hours = hours ? hours : 12;
      let minutes = isHourMark ? '00' : nextTick.getMinutes().toString().padStart(2, '0');
      let formattedTime = `${hours}:${minutes} ${ampm}`;
      ticks.push({ time: formattedTime, isHourMark: isHourMark, dateTime: nextTick });
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
            top: 15px; 
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
      
      // Calculate resize amount in minutes
      const resizeAmount = Math.round(e.detail.x / pixelsPerMinute);
      let newLength = event.length + resizeAmount;
      newLength = Math.max(5, newLength);

      event.length = newLength;
      events[index] = { ...event };
      events = [...events]; // Trigger reactivity
      dispatch('update', { events });
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
      mode = (x > rect.right - 10) ? "resize" : "move"; // Threshold for resizing
      
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

</script>
  
<div class="timeline-container" bind:this={timelineContainer}>

  {#each ticks as { time, isHourMark }, index}

    <div class="tick" style="position: relative;">
      <div class={isHourMark ? 'hour-mark' : 'minute-mark'}></div>
      {#if isHourMark}
        <div class="hour-label">&nbsp;{time}</div>
      {/if}
    </div>

  {/each}

  {#each events as event, index (event.id)}

    <div use:draggable
         style="{eventStyles[index]}"

         on:dragmove="{(e) => {
          if (e.detail.mode === 'move') {
            updateEventPosition(e, event, index);
          } else if (e.detail.mode === 'resize') {
            updateEventLength(e, event, index);
          }}}"

         on:dragend="{handleDragEnd}"
         role="gridcell"
         tabindex="0"
         aria-label="Draggable event">

      {event.label}
      
    </div>

  {/each}

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

  .hour-mark, .minute-mark {
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

  .hour-label {
    position: absolute;
    top: 0;
    background: #ffffff00; 
    color: #e7b07d;
    padding-top: 80px;
    padding-right: 10px;
    user-select: none; 
  }

</style>