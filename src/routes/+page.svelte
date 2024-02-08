<script lang="ts">
  // Initialize availability arrays for Organizer and Invitee
  const organizer_availability = Array(7)
    .fill(false)
    .map(() => Array(24).fill(false)); // 7 days, 24 hours
  const invitee_availability = Array(7)
    .fill(false)
    .map(() => Array(24).fill(false)); // 7 days, 24 hours

  // Keeps track of whether invitee has any indicated availabilities
  $: timesWork = invitee_availability
    .map((subArray) => subArray.some((elt) => elt))
    .some((elt) => elt);

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

  // Handle mouse down event for organizer
  function handleMouseDown(day: number, hour: number, organizer: boolean) {
    if (organizer) {
      toggleOrganizerAvailability(day, hour);
      lastToggle = organizer_availability[day][hour];
    } else {
      toggleInviteeAvailability(day, hour);
      lastToggle = invitee_availability[day][hour];
    }
    startDay = day;
    startHour = hour;
  }

  // Handle mouse enter event for organizer
  function handleMouseEnter(day: number, hour: number, organizer: boolean) {
    if (isMouseDown) {
      // Check if the mouse is moving horizontally
      if (day !== startDay || hour !== startHour) {
        const minDay = Math.min(day, startDay);
        const maxDay = Math.max(day, startDay);
        const minHour = Math.min(hour, startHour);
        const maxHour = Math.max(hour, startHour);
        for (let dayIter = minDay; dayIter <= maxDay; dayIter++) {
          for (let hourIter = minHour; hourIter <= maxHour; hourIter++) {
            if (organizer) {
              setOrganizerAvailability(dayIter, hourIter, lastToggle);
            } else {
              setInviteeAvailability(dayIter, hourIter, lastToggle);
            }
          }
        }
      }
    }
  }

  // Day number-to-word conversion
  function dayToWord(day: number) {
    switch (day) {
      case 0:
        return "Sunday";
      case 1:
        return "Monday";
      case 2:
        return "Tuesday";
      case 3:
        return "Wednesday";
      case 4:
        return "Thursday";
      case 5:
        return "Friday";
      case 6:
        return "Saturday";
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
    <h2 class="organizer-title">Organizer</h2>
    <div class="calendar">
      <!-- Empty top-left cell -->
      <div class="header"></div>
      {#each Array(7) as _, day}
        <div class="header">{dayToWord(day)}</div>
      {/each}

      {#each Array(24) as _, hour}
        <div class="hour-label">{hour}:00</div>
        {#each Array(7) as _, day}
          <button
            type="button"
            class="cell {organizer_availability[day][hour]
              ? 'available'
              : ''} {invitee_availability[day][hour] ? 'selected' : ''}"
            on:mousedown={() => handleMouseDown(day, hour, true)}
            on:mouseenter={() => handleMouseEnter(day, hour, true)}
          ></button>
        {/each}
      {/each}
    </div>
  </div>

  <div>
    <h2>Invitee</h2>
    <div class="calendar">
      {#each Array(7) as _, day}
        <div class="header">{dayToWord(day)}</div>
      {/each}
      <!-- Empty top-right cell -->
      <div class="header"></div>

      {#each Array(24) as _, hour}
        {#each Array(7) as _, day}
          <button
            type="button"
            class="cell {organizer_availability[day][hour]
              ? invitee_availability[day][hour]
                ? 'selected'
                : ''
              : 'unavailable'}"
            on:mousedown={() => handleMouseDown(day, hour, false)}
            on:mouseenter={() => handleMouseEnter(day, hour, false)}
          ></button>
        {/each}
        <div class="hour-label-invitee">{hour}:00</div>
      {/each}
    </div>
  </div>
</div>

<div class="overlapping-times">
  <h3>Times that work:</h3>
  {#if timesWork}
    <ul>
      {#each Array(7) as _, day}
        {#each Array(24) as _, hour}
          {#if invitee_availability[day][hour]}
            <li>{dayToWord(day)} - {hour}:00</li>
          {/if}
        {/each}
      {/each}
    </ul>
  {:else}
    <p>No times work yet! Add availabilities above based on your schedule.</p>
  {/if}
</div>

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

  .organizer-title {
    text-align: right;
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

  .hour-label-invitee {
    text-align: left;
    padding-left: 10px;
    border-left: 1px solid #ddd;
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

  .overlapping-times {
    background-color: #dbdbdb;
    width: 30%;
    margin-left: auto;
    margin-right: auto;
    margin-top: 25px;
    padding: 5px;
  }
</style>
