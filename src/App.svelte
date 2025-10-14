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
  let tooltipHeight = 0;
  let infoElement;
  let eventTooltipHeight = 0;
  let eventInfoElement;

  const grayYears = [
    2000, 2001, 2002, 2003, 2004, 2005, 2006, 2013, 2021, 2023, 2025,
  ];

  // Derivar events desde hoveredEvent
  $: events = hoveredEvent
    ? [
        { title: hoveredEvent.event, desc: hoveredEvent.event_description },
        { title: hoveredEvent.event1, desc: hoveredEvent.event1_description },
        { title: hoveredEvent.event2, desc: hoveredEvent.event2_description },
      ].filter((e) => e.title)
    : [];

  // Medir altura del tooltip superior
  $: if (infoElement) {
    tooltipHeight = infoElement.offsetHeight + 10; // margen extra
  }
  $: if (eventInfoElement) {
    eventTooltipHeight = eventInfoElement.offsetHeight + 10;
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

<div class="timeline" bind:clientWidth={width}>
  <svg {width} {height}>
    <!-- Logo -->
    <image href="static/images/logo.png" x="40" y="40" height="75" />

    <g transform="translate({margin.left}, {margin.top})">
      <Line {height} {innerWidth} {datos} />

      {#each datos as year, i}
        {#if i > 1 || i < 1}
          <g
            transform="translate({(i / (datos.length - 1)) *
              innerWidth}, {height / 2.5})"
          >
            {#if !grayYears.includes(year.year)}
              <InitiativeReport
                report={year.report}
                initiatives={year.initiatives}
                {hoveredEvent}
              />
            {/if}

            <!-- Círculo -->
            <circle
              fill={!grayYears.includes(year.year) ? "#3aadc7" : "#9d9d9c"}
              class={year ? "blink" : "normal"}
              on:mouseover={() => (hoveredEvent = year)}
              on:mouseleave={() => (hoveredEvent = null)}
            />

            <!-- Año -->
            <text
              y="25"
              text-anchor="middle"
              font-size="12"
              class:hovered={hoveredEvent && hoveredEvent.year === year.year}
              class={hoveredEvent && hoveredEvent.year === year.year
                ? "highlight-year"
                : ""}
              fill="#666"
              pointer-events="none"
              style="cursor: default;"
            >
              {year.year}
            </text>

            {#if hoveredEvent && hoveredEvent.year === year.year}
              <Tooltip
                {hoveredEvent}
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

<!-- Medición invisible -->
<div
  style="position:absolute; visibility:hidden; width:220px; pointer-events:none; white-space:pre-wrap;"
  bind:this={infoElement}
>
  {hoveredEvent?.info}
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
    max-height: 0px;
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
    gap: 20px; /* Espacio entre leyendas */
    margin: 10px auto;
    font-size: 13px;
    flex-wrap: wrap; /* Que baje a la siguiente línea si no cabe */
    justify-content: center;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 5px;
  }
</style>
