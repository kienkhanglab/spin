<template lang="pug">
.flex.flex-col(class='w-[181px] justify-center items-center')
    .text-2xl.font-bold.leading-normal {{ prize }}
</template>

<script setup lang="ts">
const props = defineProps<{
  prize: number;
}>();
</script>

<style scoped></style>
