<template>
    <g>
        <circle :r="r" :cx="cx" :cy="cy" class="state-prox-d"></circle>
    </g>
</template>

<style scoped>
    circle.state-prox-d {
        fill: #e0f2db;
        stroke: #000;
    }
</style>

<script>
    export default {
        props: {
            cx: { type: Number, default: 30/Math.SQRT2/2 },
            cy: { type: Number, default: 30/Math.SQRT2/2 },
            r: { type: Number, default: 8 },
        },
    }
</script>
