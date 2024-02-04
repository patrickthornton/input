<script lang="ts">
  // svelte imports
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";

  // fullcalendar imports
  import { Calendar } from "@fullcalendar/core";
  import interactionPlugin, { Draggable } from "@fullcalendar/interaction";
  import timeGridPlugin from "@fullcalendar/timegrid";

  // bound to div below; ! tells TS we'll define it later
  let container_elt!: HTMLElement;
  let calendar_elt!: HTMLElement;

  // tracks whether external event has been dragged onto calendar
  let dragged: boolean = false;

  // have to wait for component to be rendered to DOM before 'calendarEl' is defined
  // (rough equivalent of DOMContentLoaded in vanilla JS)
  onMount(() => {
    // create draggable event
    new Draggable(container_elt, {
      itemSelector: ".fc-event",
      eventData: function (eventEl) {
        return {
          title: eventEl.innerText,
        };
      },
    });

    // create calendar
    let calendar = new Calendar(calendar_elt, {
      plugins: [timeGridPlugin, interactionPlugin],

      // config draggability
      editable: true,
      droppable: true,
      drop: function (dropInfo) {
        dragged = true;
      },

      // config view
      initialView: "timeGridWeek",
      allDaySlot: false,

      // config toolbar
      headerToolbar: {
        left: "prev,next",
        center: "title",
        right: "timeGridWeek,timeGridDay",
      },

      // keeps calendar the exact size of browser window
      height: window.innerHeight * 0.7,
      windowResize: function (view_object) {
        view_object.view.calendar.setOption("height", window.innerHeight * 0.7);
      },
    });

    calendar.render();
  });
</script>

<div id="page">
  <div bind:this={container_elt} id="container">
    {#if !dragged}
      <p transition:fade={{ duration: 200 }}>
        <strong>Place the time below:</strong>
      </p>

      <div
        transition:fade={{ duration: 200 }}
        class="fc-event fc-h-event fc-daygrid-event fc-daygrid-block-event"
      >
        <div class="fc-event-main">Choose your time...</div>
      </div>
    {/if}
  </div>

  <div bind:this={calendar_elt} id="calendar" />
</div>

<style>
  #page {
    margin: auto;
    width: 75%;
    height: 75%;
    font-family: helvetica, sans-serif;
  }
  #container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100px;
    margin-bottom: 20px;
  }
  .fc-event {
    background-color: #3788d8;
    color: white;
    border: none;
    cursor: pointer;
    padding: 5px;
    margin: 5px;
    border-radius: 5px;
  }
</style>
