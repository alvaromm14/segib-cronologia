<script>
  import { scale } from "svelte/transition";
  import Line from "./components/Line.svelte";
  import Tooltip from "./components/Tooltip.svelte";
  import ArrowText from "./components/ArrowText.svelte";
  import { onMount } from "svelte";

  let datos = [];

  onMount(async () => {
    const res = await fetch("static/data.json"); // ya no /data/data.json
    datos = await res.json();
  });

  let width = 550;
  let height = 500;
  $: innerHeight = vertical ? height - margin.top - margin.bottom : height;

  $: margin = vertical
    ? { top: 65, right: 25, left: 25, bottom: 45 }
    : { top: 25, right: 65, left: 65 };
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

  // Evento activo
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

  // Selección de círculo
  function toggleSelection(year) {
    if (selectedEvent && selectedEvent.year === year.year) selectedEvent = null;
    else selectedEvent = year;
  }

  let tooltipActivated = false;

  // Determinar orientación y alto según ancho
  $: vertical = width < 750;
  $: height = vertical ? 800 : 500;
</script>

<div
  class="timeline"
  bind:clientWidth={width}
  on:click={() => {
    selectedEvent = null;
  }}
>
  <svg {width} {height}>
    <image
      href="static/images/logo.png"
      x="0"
      y="0"
      height={vertical ? "50" : "75"}
    />

    <g transform="translate({margin.left}, {margin.top})">
      <Line height={innerHeight} {innerWidth} {datos} {vertical} />
      {#if !selectedEvent && !hoveredEvent}
        <ArrowText {innerWidth} {height} {datos} {vertical} />
      {/if}

      {#each datos as year, i}
        <g
          transform={vertical
            ? `translate(${innerWidth / 2}, ${(i / (datos.length - 1)) * innerHeight})`
            : `translate(${(i / (datos.length - 1)) * innerWidth}, ${innerHeight / 2.5})`}
        >
          <!-- Círculo -->
          {#if year.year !== 2001}
            <circle
              r="7.5"
              fill={!grayYears.includes(year.year) ? "#589ea4" : "#ecba56"}
              stroke="black"
              stroke-width="0.4"
              class:blink={!tooltipActivated}
              on:mouseover={() => {
                hoveredEvent = year;
                selectedEvent = null;
                tooltipActivated = true;
              }}
              on:mouseleave={() => (hoveredEvent = null)}
              on:click={(e) => {
                e.stopPropagation();
                toggleSelection(year);
                tooltipActivated = true;
              }}
            />
          {/if}

          <!-- Año -->
          {#if !(width >= 750 && width <= 1000 && year.year % 2 !== 0) && year.year !== 2001}
            <text
              x={vertical ? 14 : 0}
              y={vertical ? 5 : 23}
              text-anchor={vertical ? "start" : "middle"}
              font-size="12"
              class:hovered={activeEvent && activeEvent.year === year.year}
              class={activeEvent && activeEvent.year === year.year
                ? "highlight-year"
                : ""}
              fill="#666"
              pointer-events="none"
              style="cursor: default; user-select: none"
            >
              {year.year}
            </text>
          {/if}

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
              {vertical}
            />
          {/if}
        </g>
      {/each}
    </g>

    <image
      href="static/images/logos.jpg"
      x={0}
      y={vertical ? height - 20 : height - 40}
      width={vertical ? "150" : "250"}
    />
  </svg>
</div>

<div
  style="position:absolute; visibility:hidden; width:220px; pointer-events:none; white-space:pre-wrap;"
  bind:this={infoElement}
>
  {activeEvent?.info}
</div>

<style>
  :global(body) {
    overflow-y: hidden;
  }
  .timeline {
    position: relative;
    max-width: 1500px;
    margin: 0 auto;
    overflow: hidden;
  }

  .timeline circle {
    cursor: pointer;
    transition: transform 0.3s;
    filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
  }

  .timeline circle:hover {
    transform: scale(1.2);
  }

  @keyframes pulse {
    0%,
    100% {
      transform: scale(1);
      opacity: 1;
    }
    50% {
      transform: scale(1.2);
      opacity: 0.85;
    }
  }

  .blink {
    animation: pulse 1.5s ease-in-out infinite;
  }

  .highlight-year {
    font-weight: bold;
    fill: #222;
  }
</style>
