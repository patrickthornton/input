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

  // Keeps track of military time setting
  let military = false;

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
  let lastDay = 0;
  let lastHour = 0;

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
    lastDay = day;
    lastHour = hour;
  }

  // Handle mouse enter event for organizer
  function handleMouseEnter(day: number, hour: number, organizer: boolean) {
    function dragSelect(cornerDay: number, cornerHour: number, value: boolean) {
      const minDay = Math.min(cornerDay, startDay);
      const maxDay = Math.max(cornerDay, startDay);
      const minHour = Math.min(cornerHour, startHour);
      const maxHour = Math.max(cornerHour, startHour);
      for (let dayIter = minDay; dayIter <= maxDay; dayIter++) {
        for (let hourIter = minHour; hourIter <= maxHour; hourIter++) {
          if (organizer) {
            setOrganizerAvailability(dayIter, hourIter, value);
          } else {
            setInviteeAvailability(dayIter, hourIter, value);
          }
        }
      }
    }
    if (isMouseDown) {
      // This first call allows for retractions of drag selects
      dragSelect(lastDay, lastHour, !lastToggle);
      // This actually selects the desired boxes
      dragSelect(day, hour, lastToggle);
      lastDay = day;
      lastHour = hour;
    }
  }

  // Day number-to-string conversion
  function dayToString(day: number) {
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

  // Convert hour to string
  function hourToString(hour: number, military: boolean) {
    if (military) {
      return `${hour}:00`;
    } else {
      if (hour === 0) {
        return "12 am";
      } else if (hour < 12) {
        return `${hour} am`;
      } else if (hour === 12) {
        return "12 pm";
      } else {
        return `${hour - 12} pm`;
      }
    }
  }

  // Handle military time button functionality
  function militaryButton() {
    military = !military;
  }

  // Calendar titles
  let leftTitle = "Organizer";
  let rightTitle = "Invitee 1";
  let relays = 0;

  // Handle relay button functionality
  function relayButton() {
    function updateTitles() {
      relays += 1;
      leftTitle = `Invitee ${relays}`;
      rightTitle = `Invitee ${relays + 1}`;
    }
    for (let day = 0; day < 7; day++) {
      for (let hour = 0; hour < 24; hour++) {
        organizer_availability[day][hour] = invitee_availability[day][hour];
        invitee_availability[day][hour] = false;
      }
    }
    updateTitles();
  }

  // Handle reset button functionality
  function resetButton() {
    for (let day = 0; day < 7; day++) {
      for (let hour = 0; hour < 24; hour++) {
        organizer_availability[day][hour] = false;
        invitee_availability[day][hour] = false;
      }
    }
    leftTitle = "Organizer";
    rightTitle = "Invitee 1";
    relays = 0;
  }
</script>

<svelte:document
  on:mousedown={updateIsMouseDown}
  on:mouseover={updateIsMouseDown}
  on:mouseup={updateIsMouseDown}
/>

<div class="calendar-container">
  <div>
    <h2 class="organizer-title">{leftTitle}</h2>
    <div class="calendar">
      <!-- Empty top-left cell -->
      <div class="header"></div>
      {#each Array(7) as _, day}
        <div class="header">{dayToString(day)}</div>
      {/each}

      {#each Array(24) as _, hour}
        <div class="hour-label">{hourToString(hour, military)}</div>
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

  <div class="buttons">
    <button
      type="button"
      class="button military-button"
      on:click={militaryButton}
    >
      {military ? "Switch to 12-hour" : "Switch to 24-hour"}
    </button>
    <button type="button" class="button relay-button" on:click={relayButton}>
      <strong>Relay</strong>
    </button>
    <button type="button" class="button reset-button" on:click={resetButton}>
      Reset
    </button>
  </div>

  <div>
    <h2 class="invitee-title">{rightTitle}</h2>
    <div class="calendar">
      {#each Array(7) as _, day}
        <div class="header">{dayToString(day)}</div>
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
        <div class="hour-label-invitee">{hourToString(hour, military)}</div>
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
            <li>{dayToString(day)} - {hourToString(hour, military)}</li>
          {/if}
        {/each}
      {/each}
    </ul>
  {:else}
    <p>No times work yet! Add availabilities above based on your schedule.</p>
  {/if}
  <p>
    When finished, hit the <strong>Relay</strong> button to relay your availabilities
    to the next invitee.
  </p>
</div>

<div class="overlapping-times">
  <p>
    Note: you are looking at a demo version of this application that allows you
    to simulate the behavior of both the organizer and each of the invitees! In
    practice, each invitee would only see the availabilities of the invitee
    before them; but we felt it best to allow you to see the full picture for
    demonstration purposes.
  </p>
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

  .buttons {
    display: flex;
    flex-flow: column;
    justify-content: center;
    gap: 20px;
  }

  .button {
    height: 30px;
  }

  .relay-button {
    height: 45px;
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
