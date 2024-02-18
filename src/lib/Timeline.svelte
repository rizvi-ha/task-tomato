<script>

    export let length;
  
    function generateTimelineHours(length) {
      const now = new Date();
      const roundedMinutes = Math.floor(now.getMinutes() / 30) * 30;
      now.setMinutes(roundedMinutes);
      now.setSeconds(0);
      now.setMilliseconds(0);
  
      if (now.getMinutes() === 60) {
        now.setHours(now.getHours() + 1);
        now.setMinutes(0);
      }
  
      let hours = [];
      for (let i = 0; i < length; i++) {
        let nextHour = new Date(now.getTime() + i * 60 * 60 * 1000);
        let formattedTime = nextHour.getHours().toString().padStart(2, '0') + ':' + nextHour.getMinutes().toString().padStart(2, '0');
        hours.push(formattedTime);
      }
      return hours;
    }
  
    $: hours = generateTimelineHours(length);
    
</script>
  
  <div class="timeline-container">
    {#each hours as hour, index}
      <div class="hour-mark">&nbsp;{hour}:00</div>
    {/each}
  </div>
  
  <style>
    .timeline-container {
      width: 100%;
      height: 100px;
      white-space: nowrap;
      border-top: 2px solid #eab804;
      position: relative;
    }
    .hour-mark {
      display: inline-block;
      height: 100px;
      width: 200px;
      border-left: 3px solid #eab804;
      text-align: left;
    }
  </style>