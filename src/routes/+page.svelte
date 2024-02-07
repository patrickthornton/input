<script>
  // Initialize availability arrays for Organizer and Invitee
  let availability1 = Array(7)
    .fill()
    .map(() => Array(24).fill(false)); // 7 days, 24 hours
  let availability2 = Array(7)
    .fill()
    .map(() => Array(24).fill(false)); // 7 days, 24 hours
  let selectedSlots = Array(7)
    .fill()
    .map(() => Array(24).fill(false)); // 7 days, 24 hours

  // Track mouse state and starting indices for selection
  let isMouseDown = false;
  let startDayIndex = null;
  let startHourIndex = null;

  // Toggle availability for a specific hour in a specific day for Organizer
  function toggleAvailability1(dayIndex, hourIndex) {
    availability1[dayIndex][hourIndex] = !availability1[dayIndex][hourIndex];
    if (!availability1[dayIndex][hourIndex]) {
      selectedSlots[dayIndex][hourIndex] = false;
    }
    updateAvailability2(dayIndex, hourIndex);
  }

  // Update availability for Invitee based on Organizer
  function updateAvailability2(dayIndex, hourIndex) {
    if (availability1[dayIndex][hourIndex]) {
      availability2[dayIndex][hourIndex] = true;
    } else {
      availability2[dayIndex][hourIndex] = false;
      selectedSlots[dayIndex][hourIndex] = false;
    }
  }

  // Toggle availability for a specific hour in a specific day for Invitee
  function toggleAvailability2(dayIndex, hourIndex) {
    if (availability2[dayIndex][hourIndex]) {
      selectedSlots[dayIndex][hourIndex] = !selectedSlots[dayIndex][hourIndex];
    }
  }

  // Check for overlapping times
  function getOverlappingTimes() {
    let overlappingTimes = [];
    for (let dayIndex = 0; dayIndex < 7; dayIndex++) {
      for (let hourIndex = 0; hourIndex < 24; hourIndex++) {
        if (
          availability1[dayIndex][hourIndex] &&
          availability2[dayIndex][hourIndex]
        ) {
          overlappingTimes.push({ dayIndex, hourIndex });
        }
      }
    }
    return overlappingTimes;
  }

  // Handle mouse down event
  function handleMouseDown(dayIndex, hourIndex) {
    isMouseDown = true;
    startDayIndex = dayIndex;
    startHourIndex = hourIndex;
    toggleAvailability1(dayIndex, hourIndex);
  }

  // Handle mouse enter event
  function handleMouseEnter(dayIndex, hourIndex) {
    if (isMouseDown) {
      // Check if the mouse is moving vertically
      if (dayIndex !== startDayIndex) {
        const minDayIndex = Math.min(dayIndex, startDayIndex);
        const maxDayIndex = Math.max(dayIndex, startDayIndex);
        for (let i = minDayIndex; i <= maxDayIndex; i++) {
          toggleAvailability1(i, hourIndex);
        }
      }
    }
  }

  // Handle mouse up event
  function handleMouseUp() {
    isMouseDown = false;
    startDayIndex = null;
    startHourIndex = null;
  }
</script>

<div class="calendar-container">
  <div>
    <h2>Organizer</h2>
    <div class="calendar">
      <div class="header"></div>
      <!-- Empty top-left cell -->
      <div class="header">Sunday</div>
      <div class="header">Monday</div>
      <div class="header">Tuesday</div>
      <div class="header">Wednesday</div>
      <div class="header">Thursday</div>
      <div class="header">Friday</div>
      <div class="header">Saturday</div>

      {#each Array(24) as _, hour}
        <div class="hour-label">{hour}:00</div>
        {#each Array(7) as _, day}
          <div
            class="cell {availability1[day][hour]
              ? 'available'
              : ''} {selectedSlots[day][hour] ? 'selected' : ''}"
            on:click={() => toggleAvailability1(day, hour)}
          ></div>
        {/each}
      {/each}
    </div>
  </div>

  <div>
    <h2>Invitee</h2>
    <div class="calendar">
      <div class="header"></div>
      <!-- Empty top-left cell -->
      <div class="header">Sunday</div>
      <div class="header">Monday</div>
      <div class="header">Tuesday</div>
      <div class="header">Wednesday</div>
      <div class="header">Thursday</div>
      <div class="header">Friday</div>
      <div class="header">Saturday</div>

      {#each Array(24) as _, hour}
        <div class="hour-label">{hour}:00</div>
        {#each Array(7) as _, day}
          <div
            class="cell {availability2[day][hour]
              ? selectedSlots[day][hour]
                ? 'selected'
                : ''
              : 'unavailable'}"
            on:click={() => toggleAvailability2(day, hour)}
          ></div>
        {/each}
      {/each}
    </div>
  </div>
</div>

{#if getOverlappingTimes().length > 0}
  <h3>Overlapping Times:</h3>
  <ul>
    {#each getOverlappingTimes() as { dayIndex, hourIndex }}
      <li>{dayIndex} - {hourIndex}</li>
    {/each}
  </ul>
{/if}

<style>
  .calendar-container {
    display: flex;
    justify-content: space-between;
    gap: 20px;
  }

  .calendar {
    display: grid;
    grid-template-columns: repeat(
      8,
      1fr
    ); /* 7 days + 1 column for hour labels */
    text-align: center;
  }

  .header {
    font-weight: bold;
    background-color: #f4f4f4;
    padding: 10px;
  }

  .hour-label {
    text-align: right;
    padding-right: 10px;
    border-right: 1px solid #ddd;
  }

  .cell {
    padding: 5px;
    border: 1px solid #ddd;
    cursor: pointer;
  }

  .available {
    background-color: #fcf87a;
  }

  .selected {
    background-color: #a0ffa0;
  }

  .unavailable {
    background-color: #ddd;
    cursor: default;
  }

  .overlapping-time {
    background-color: #ff8080;
  }
</style>
