<script>

  export let length;

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
      ticks.push({ time: formattedTime, isHourMark: isHourMark });
    }
    return ticks;
  }

  $: ticks = generateTimeline(length);

</script>
  
<div class="timeline-container">
  {#each ticks as { time, isHourMark }, index}
    <div class="tick" style="position: relative;">
      <div class={isHourMark ? 'hour-mark' : 'minute-mark'}></div>
      {#if isHourMark}
        <div class="hour-label">&nbsp;{time}</div>
      {/if}
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
  }

  .tick {
    flex: 1;
    position: relative;
  }

  .hour-mark, .minute-mark {
    position: absolute;
    width: 100%;
    left: 0;
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
  }
  
</style>