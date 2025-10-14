<script>
    import { scale } from "svelte/transition";
    import { onMount } from "svelte";

    export let hoveredEvent;
    export let i;
    export let datos;
    export let infoElement;
    export let eventInfoElement;
    export let tooltipHeight = 0;
    export let eventTooltipHeight = 0;

    // Derivamos los eventos
    $: events = hoveredEvent
        ? [
              {
                  title: hoveredEvent.event,
                  desc: hoveredEvent.event_description,
              },
              {
                  title: hoveredEvent.event1,
                  desc: hoveredEvent.event1_description,
              },
              {
                  title: hoveredEvent.event2,
                  desc: hoveredEvent.event2_description,
              },
          ].filter((e) => e.title)
        : [];

    const xSide = i / (datos.length - 1);

    // Ruta de la imagen
    $: imageSrc = hoveredEvent?.report
        ? `static/images/${hoveredEvent.report}.png`
        : null;

    // Preload de todas las imágenes de reports
    onMount(() => {
        datos.forEach((d) => {
            if (d.report) {
                const img = new Image();
                img.src = `static/images/${d.report}.png`;
            }
        });
    });
</script>

{#if hoveredEvent}
    <!-- Tooltip abajo (events) -->
    {#if events.length}
        <g class="tooltip-group" transition:scale={{ duration: 400 }}>
            <line
                y1="35"
                y2={51 + eventTooltipHeight}
                stroke="#9d9d9c"
                stroke-width="2"
            />

            <foreignObject
                x={xSide < 0.8 ? 8 : -228}
                y="65"
                width="250"
                height="200"
                pointer-events="none"
            >
                <div
                    xmlns="http://www.w3.org/1999/xhtml"
                    class="tooltip"
                    style="text-align: {xSide >= 0.8 ? 'right' : 'left'};"
                    bind:this={eventInfoElement}
                >
                    {#each events as e, index}
                        <div
                            style="font-weight: normal; padding-bottom: 4px;"
                            transition:scale={{
                                duration: index > 0 ? 400 : 0,
                                delay: index * 150,
                            }}
                        >
                            <span style="font-weight: bold;">{e.title}</span>
                            {e.desc ? ` ${e.desc}` : ""}
                        </div>

                        {#if index < events.length - 1}
                            <hr
                                class="tooltip-divider"
                                transition:scale={{
                                    duration: 400,
                                    delay: index * 200 + 50,
                                }}
                            />
                        {/if}
                    {/each}
                </div>
            </foreignObject>
        </g>
    {/if}

    <!-- Tooltip arriba (info) -->
    {#if hoveredEvent.info}
        <g class="tooltip-group" transition:scale={{ duration: 400 }}>
            <line
                y1="-20"
                y2={-tooltipHeight - 30}
                stroke="#3aadc7"
                stroke-width="2"
            />

            {#if hoveredEvent.report}
                <rect
                    x={xSide >= 0.8 ? "6" : "-29"}
                    y="-158"
                    width="20"
                    height="16"
                    fill="white"
                    rx="4"
                    stroke="#3aadc7"
                />
                <text
                    y="-146"
                    x={xSide >= 0.8 ? "16" : "-19"}
                    text-anchor="middle"
                    font-size="0.7em"
                    fill="#3aadc7"
                    font-weight="bold"
                >
                    {hoveredEvent.report}
                </text>
            {/if}

            <foreignObject
                x={xSide < 0.8 ? 8 : -228}
                y={-tooltipHeight - 32}
                width="250"
                height={tooltipHeight}
            >
                <div
                    xmlns="http://www.w3.org/1999/xhtml"
                    class="tooltip info-tooltip"
                    style="text-align: {xSide >= 0.8 ? 'right' : 'left'};"
                    pointer-events="none"
                    bind:this={infoElement}
                >
                    {@html hoveredEvent.info.replace(
                        /<b>/g,
                        '<b style="font-weight:bold; display:inline;">',
                    )}
                </div>
            </foreignObject>

            <!-- Imagen pre-cargada -->
            {#if imageSrc}
                <foreignObject
                    x={xSide < 0.8 ? -95 : 5}
                    y="-135"
                    width="90"
                    height="120"
                >
                    <div
                        xmlns="http://www.w3.org/1999/xhtml"
                        style="display:flex; justify-content:center; align-items:center;"
                        transition:scale={{ duration: 400 }}
                    >
                        <img
                            src={imageSrc}
                            alt={`${hoveredEvent.report}`}
                            style="width: 80px; height: auto; border: 1px solid #ccc;"
                        />
                    </div>
                </foreignObject>
            {/if}
        </g>
    {/if}
{/if}

<style>
    .tooltip {
        color: #9d9d9c;
        font-size: 13px;
        max-width: 220px;
        word-wrap: break-word;
        background: none;
        padding: 0;
        user-select: none;
    }

    .tooltip-divider {
        border: none;
        border-top: 1px solid #ccc;
        margin: 4px 0;
    }

    .info-tooltip {
        color: #3aadc7;
        display: block;
        flex-direction: column;
        justify-content: flex-end;
    }
</style>
