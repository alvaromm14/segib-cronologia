<script>
    export let height;
    export let innerWidth;
    export let datos;
</script>

<line
    x1="0"
    y1={height / 2.5}
    x2={innerWidth}
    y2={height / 2.5}
    stroke="#9d9d9c"
    stroke-width="1.5"
/>

<line
    x1={(7 / (datos.length - 1)) * innerWidth}
    y1={height / 2.5}
    x2={(12 / (datos.length - 1)) * innerWidth}
    y2={height / 2.5}
    stroke="#3aadc7"
    stroke-width="2"
/>

<line
    x1={(14 / (datos.length - 1)) * innerWidth}
    y1={height / 2.5}
    x2={(20 / (datos.length - 1)) * innerWidth}
    y2={height / 2.5}
    stroke="#3aadc7"
    stroke-width="2"
/>
