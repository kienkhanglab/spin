<template lang="pug">
div.flex.justify-center
    button.relative.flex.justify-center.items-center.custom-shadow(class='w-[80px] h-[80px] border-[3px] border-double font-bold overflow-hidden rounded-full disabled:cursor-not-allowed' @click='$emit("start")' v-show='!isSpinning') SPIN    
    button.relative.flex.justify-center.items-center.custom-shadow(class='w-[80px] h-[80px] border-[3px] border-double font-bold overflow-hidden rounded-full disabled:cursor-not-allowed' @click='$emit("stop")' v-show='isSpinning') STOP

</template>

<script setup lang="ts">
defineProps<{
    isSpinning: boolean
}>()

const emit = defineEmits<{
    (e: 'start'): void
    (e: 'stop'): void
}>()
</script>

<style scoped lang="stylus">
.custom-shadow
    box-shadow: 0px 0px 30px 0.5px #F91880;
</style>
    