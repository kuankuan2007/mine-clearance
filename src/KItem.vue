<template>
    <div class="item" :class="{
        used: map[x][y].tag !== undefined,
        'is-mind': map[x][y].tag === 2,
    }">
        <div class="value" v-if="map[x][y].tag === 1 && map[x][y].showValue > 0 && !map[x][y].value">
            {{ map[x][y].showValue }}
        </div>
    </div>
</template>
<script setup>
import { computed } from "vue"

const props = defineProps({
    x: Number,
    y: Number,
    map: Object
})
</script>
<style scoped lang="scss">
.item {
    $borderWidth: 2px;
    width: calc(var(--size) - 2 * $borderWidth);
    height: calc(var(--size) - 2 * $borderWidth);
    border: $borderWidth solid var(--color);
    border-radius: 3 * $borderWidth;
    color: var(--color);
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: var(--color);
    font-size: calc(var(--size) / 1.6);
    cursor: pointer;

    &.used {
        cursor: default;
        background-color: transparent;

        &.is-mind {
            background-color: var(--mine-color);
            border-color: var(--mine-color);
        }
    }
}
</style>