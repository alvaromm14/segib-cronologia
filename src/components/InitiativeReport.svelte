<script>
  export let report;
  export let initiatives;
  export let hoveredEvent;

  import { fade, scale } from "svelte/transition";

  let textWidth = 0;
  let textElement;

  $: if (textElement) {
    textWidth = textElement.getBBox().width;
  }

  // Si el texto contiene un espacio (por ejemplo, "3.500 iniciativas"),
  // separamos las dos partes
  $: parts = initiatives ? initiatives.split(" ") : [];
</script>

<g>
  {#if initiatives && !hoveredEvent}
    <g transition:scale={{ duration: 600, opacity: 0 }}>
      <!-- Texto initiatives -->
      <text
        bind:this={textElement}
        y={parts.length > 1 ? "-50" : "-37"}
        text-anchor="middle"
        font-size={initiatives === "3.500 iniciativas"
          ? "0.7rem"
          : initiatives === "5.000"
            ? "0.75rem"
            : initiatives === "7.500"
              ? "0.8rem"
              : initiatives === "9.000"
                ? "0.85rem"
                : "0.9rem"}
        fill="#212c55"
        font-weight="bold"
      >
        {#if parts.length > 1}
          <tspan x="0">{parts[0]}</tspan>
          <tspan x="0" dy="1.1em">{parts.slice(1).join(" ")}</tspan>
        {:else}
          {initiatives}
        {/if}
      </text>
    </g>
  {/if}

  {#if report && !hoveredEvent}
    <g transition:scale={{ duration: 600, opacity: 0 }}>
      <!-- Fondo cuadrado para report -->
      <rect x="-10" y="-27.5" width="20" height="16" fill="#3aadc7" rx="4" />
      <!-- Número de report -->
      <text
        y="-16"
        text-anchor="middle"
        font-size="0.7em"
        fill="#fff"
        font-weight="bold"
        pointer-events="none"
      >
        {report}
      </text>
    </g>
  {/if}
</g>
