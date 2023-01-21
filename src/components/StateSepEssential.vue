<template>
    <g>
        <circle :r="r" :cx="cx" :cy="cy" class="state-sep-e"></circle>
    </g>
</template>

<style scoped>
    
    
    circle.state-sep-e {
        fill: #F8623A;
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
