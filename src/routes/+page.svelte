<script lang="ts">
    import { onMount } from "svelte";
    import { Calendar } from "@fullcalendar/core";
    import timeGridPlugin from "@fullcalendar/timegrid";

    let height_fix: number = 16;

    // bound to div below; ! tells TS we'll define it later
    let calendarEl!: HTMLElement;

    // have to wait for component to be rendered to DOM before 'calendarEl' is defined
    // (rough equivalent of DOMContentLoaded in vanilla JS)
    onMount(() => {
        let calendar = new Calendar(calendarEl, {
            plugins: [timeGridPlugin],
            initialView: "timeGridWeek",
            headerToolbar: {
                left: "prev,next",
                center: "title",
                right: "timeGridWeek,timeGridDay",
            },
            height: window.innerHeight - height_fix,
            // keeps calendar the exact size of browser window
            windowResize: function (view_object) {
                view_object.view.calendar.setOption(
                    "height",
                    window.innerHeight - height_fix,
                );
            },
        });

        calendar.render();
    });
</script>

<div bind:this={calendarEl} />
