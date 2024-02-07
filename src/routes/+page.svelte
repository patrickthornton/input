<script lang="ts">
  // Initialize availability arrays for Organizer and Invitee
  let organizer_availability = Array(7)
    .fill(false)
    .map(() => Array(24).fill(false)); // 7 days, 24 hours
  let invitee_availability = Array(7)
    .fill(false)
    .map(() => Array(24).fill(false)); // 7 days, 24 hours

  // Toggle availability for a specific hour in a specific day for Organizer
  function toggleOrganizerAvailability(day: number, hour: number) {
    organizer_availability[day][hour] = !organizer_availability[day][hour];
    updateInviteeAvailability(day, hour);
  }

  // Set availability for a specific hour in a specific day for Organizer; used in drag selects
  function setOrganizerAvailability(day: number, hour: number, value: boolean) {
    organizer_availability[day][hour] = value;
    updateInviteeAvailability(day, hour);
  }

  // Update invitee availability based on organizer availability
  function updateInviteeAvailability(day: number, hour: number) {
    if (!organizer_availability[day][hour]) {
      invitee_availability[day][hour] = false;
    }
  }

  // Toggle availability for a specific hour in a specific day for Invitee
  function toggleInviteeAvailability(day: number, hour: number) {
    if (organizer_availability[day][hour]) {
      invitee_availability[day][hour] = !invitee_availability[day][hour];
    }
  }

  // Set availability for a specific hour in a specific day for Invitee; used in drag selects
  function setInviteeAvailability(day: number, hour: number, value: boolean) {
    if (organizer_availability[day][hour]) {
      invitee_availability[day][hour] = value;
    }
  }

  // Check for overlapping times
  function getOverlappingTimes() {
    let overlappingTimes = [];
    for (let day = 0; day < 7; day++) {
      for (let hour = 0; hour < 24; hour++) {
        if (organizer_availability[day][hour]) {
          overlappingTimes.push({ dayIndex: day, hourIndex: hour });
        }
      }
    }
    return overlappingTimes;
  }

  // Track mouse state and starting indices for selection
  let isMouseDown = false;
  let lastToggle = false;
  let startDay = 0;
  let startHour = 0;

  // This bit of code has been adapted from https://stackoverflow.com/questions/322378/javascript-check-if-mouse-button-down
  // Tracks whether left mouse button is clicked; see bindings on svelte:document below
  function updateIsMouseDown(event: MouseEvent) {
    isMouseDown = (event.buttons & 1) === 1;
  }

  // Handle mouse down event
  function handleMouseDown(day: number, hour: number) {
    toggleOrganizerAvailability(day, hour);
    lastToggle = organizer_availability[day][hour];
    startDay = day;
    startHour = hour;
  }

  // Handle mouse enter event
  function handleMouseEnter(day: number, hour: number) {
    if (isMouseDown) {
      // Check if the mouse is moving horizontally
      if (day !== startDay || hour !== startHour) {
        const minDay = Math.min(day, startDay);
        const maxDay = Math.max(day, startDay);
        const minHour = Math.min(hour, startHour);
        const maxHour = Math.max(hour, startHour);
        for (let dayIter = minDay; dayIter <= maxDay; dayIter++) {
          for (let hourIter = minHour; hourIter <= maxHour; hourIter++) {
            setOrganizerAvailability(dayIter, hourIter, lastToggle);
          }
        }
      }
    }
  }
</script>

<svelte:document
  on:mousedown={updateIsMouseDown}
  on:mouseover={updateIsMouseDown}
  on:mouseup={updateIsMouseDown}
/>

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
          <button
            type="button"
            class="cell {organizer_availability[day][hour]
              ? 'available'
              : ''} {invitee_availability[day][hour] ? 'selected' : ''}"
            on:mousedown={() => handleMouseDown(day, hour)}
            on:mouseenter={() => handleMouseEnter(day, hour)}
          ></button>
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
          <button
            type="button"
            class="cell {organizer_availability[day][hour]
              ? invitee_availability[day][hour]
                ? 'selected'
                : ''
              : 'unavailable'}"
            on:click={() => toggleInviteeAvailability(day, hour)}
          ></button>
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
</style>
