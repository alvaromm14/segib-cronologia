<script>
  import { scale } from "svelte/transition";
  import InitiativeReport from "./components/InitiativeReport.svelte";
  import Line from "./components/Line.svelte";
  import datos from "./data/data.js";
  import Tooltip from "./components/Tooltip.svelte";

  let width = 460,
    height = 460;

  const margin = { top: 25, right: 65, left: 65 };
  $: innerWidth = width - margin.left - margin.right;

  let hoveredEvent = null;
  let selectedEvent = null;
  let tooltipHeight = 0;
  let infoElement;
  let eventTooltipHeight = 0;
  let eventInfoElement;

  const grayYears = [
    2000, 2001, 2002, 2003, 2004, 2005, 2006, 2013, 2021, 2023, 2025,
  ];

  // Evento activo (hover o click)
  $: activeEvent = hoveredEvent || selectedEvent;

  // Derivar events desde activeEvent
  $: events = activeEvent
    ? [
        { title: activeEvent.event, desc: activeEvent.event_description },
        { title: activeEvent.event1, desc: activeEvent.event1_description },
        { title: activeEvent.event2, desc: activeEvent.event2_description },
      ].filter((e) => e.title)
    : [];

  // Medir alturas
  $: if (infoElement) tooltipHeight = infoElement.offsetHeight + 10;
  $: if (eventInfoElement)
    eventTooltipHeight = eventInfoElement.offsetHeight + 10;

  // Función para click en círculo
  function toggleSelection(year) {
    if (selectedEvent && selectedEvent.year === year.year) {
      selectedEvent = null; // desactiva si clicas el mismo
    } else {
      selectedEvent = year; // selecciona uno nuevo
    }
  }
</script>

<h1>15 Informes Sur-Sur en 25 años de Cooperación al Desarrollo</h1>

<div class="legend">
  <div class="legend-item">
    <svg width="16" height="16">
      <circle cx="8" cy="8" r="6" fill="#3aadc7" />
    </svg>
    <span>Informe de la Cooperación Sur-Sur y Triangular en Iberoamérica</span>
  </div>
  <div class="legend-item">
    <svg width="16" height="16">
      <circle cx="8" cy="8" r="6" fill="#9d9d9c" />
    </svg>
    <span
      >Agenda Internacional para el Desarrollo y Agenda Global CSS y Triangular</span
    >
  </div>
</div>

<div
  class="timeline"
  bind:clientWidth={width}
  on:click={() => (selectedEvent = null)}
>
  <svg {width} {height}>
    <image href="static/images/logo.png" x="40" y="40" height="75" />

    <g transform="translate({margin.left}, {margin.top})">
      <Line {height} {innerWidth} {datos} />

      {#each datos as year, i}
        {#if i > 1 || i < 1}
          <g
            transform="translate({(i / (datos.length - 1)) *
              innerWidth}, {height / 2.5})"
          >
            {#if !grayYears.includes(year.year) && !activeEvent}
              <InitiativeReport
                report={year.report}
                initiatives={year.initiatives}
              />
            {/if}

            <!-- Círculo interactivo -->
            <circle
              fill={!grayYears.includes(year.year) ? "#3aadc7" : "#9d9d9c"}
              class={year ? "blink" : "normal"}
              on:mouseover={() => {
                hoveredEvent = year;
                selectedEvent = null; // olvida el tooltip fijado
              }}
              on:mouseleave={() => (hoveredEvent = null)}
              on:click={(e) => {
                e.stopPropagation();
                toggleSelection(year);
              }}
            />

            <!-- Año -->
            <text
              y="25"
              text-anchor="middle"
              font-size="12"
              class:hovered={activeEvent && activeEvent.year === year.year}
              class={activeEvent && activeEvent.year === year.year
                ? "highlight-year"
                : ""}
              fill="#666"
              pointer-events="none"
              style="cursor: default;"
            >
              {year.year}
            </text>

            <!-- Tooltip -->
            {#if activeEvent && activeEvent.year === year.year}
              <Tooltip
                hoveredEvent={activeEvent}
                {i}
                {datos}
                bind:infoElement
                bind:eventInfoElement
                {tooltipHeight}
                {eventTooltipHeight}
              />
            {/if}
          </g>
        {/if}
      {/each}
    </g>
  </svg>
</div>

<div
  style="position:absolute; visibility:hidden; width:220px; pointer-events:none; white-space:pre-wrap;"
  bind:this={infoElement}
>
  {activeEvent?.info}
</div>

<style>
  h1 {
    text-align: center;
    margin: 10px auto;
    color: #212c55;
    font-family: "Helvetica Neue", Arial, sans-serif;
    font-size: 24px;
    font-weight: bold;
    letter-spacing: 1px;
    border-bottom: 2px solid #3aadc7;
    display: inline-block;
    padding-bottom: 5px;
    display: block;
    max-width: 1500px;
  }

  .timeline {
    position: relative;
    max-width: 1500px;
    margin: 0 auto;
  }

  .timeline circle {
    cursor: pointer;
    transition: transform 0.2s;
  }
  .timeline circle:hover {
    transform: scale(1.2);
  }

  @keyframes blinkAnimation {
    0%,
    100% {
      opacity: 1;
      r: 7.5;
    }
    50% {
      opacity: 0.6;
      r: 9;
    }
  }
  .blink {
    animation: blinkAnimation 1s ease-in-out 3 forwards;
  }

  .highlight-year {
    font-weight: bold;
    fill: #000;
  }

  .legend {
    display: flex;
    align-items: center;
    gap: 20px;
    margin: 10px auto;
    font-size: 13px;
    flex-wrap: wrap;
    justify-content: center;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 5px;
  }
</style>
