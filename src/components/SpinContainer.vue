<template lang="pug">
div
    SpinMachine(
        @fire-animation='fireAnimation'
        @update-result='openResultModal'
    )
    .absolute.inset-0.flex.items-center.justify-center.h-full(v-if='showAnimation')
      Vue3Lottie.shrink-0(
        ref='lottie' 
        :animationData="animtion" 
        :height="512" 
        :width="512" 
        :speed='3' 
        :loop='false' 
        @on-complete='onComplete'
      )


</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Vue3Lottie, type AnimationItem } from 'vue3-lottie'
import Swal from 'sweetalert2'
import animtion from '../constants/animation.json'

import 'sweetalert2/dist/sweetalert2.min.css';

import SpinMachine from "./SpinMachine.vue";

const lottie = ref<AnimationItem>()
const showAnimation = ref(false)

function fireAnimation(){
  // Call open lottie
  showAnimation.value = true
  lottie.value?.play()
}
function onComplete(){
  // Close Lottie
  lottie.value?.stop()
  showAnimation.value = false
}

function openResultModal(gift: number){
    console.log({gift})
    Swal.fire({
        title:'Congratulations',
        text:`You get ${gift}`
    })
}

</script>

<style scoped></style>
