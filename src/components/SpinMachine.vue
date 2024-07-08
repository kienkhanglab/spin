<template lang="pug">
.flex-1.flex.justify-center.items-center.h-full(class='max-h-[420px] mt-[30px]')
  .relative.w-max.flex.flex-col.items-center
    // Item to spin
    .machine.spin-view.border.border-yellow(class='overflow-hidden')
      // Spin when loading
      .machine-container(v-show='isSpinning' :class="cn({'spinning' : isSpinning})" :style='elStyle')
        SpinItem.spin-item(
          v-for='prize in cloneItems' 
          :key='prize'
          :prize='prize' 
          :class="cn({ 'loading-shadow': isSpinning })"
        ) 
      // Spin when no loading
      .machine-container(v-show='!isSpinning' :style='elStyle')
        SpinItem.spin-item(
          v-for='(prize, idx) in items' 
          :key='prize'
          :prize='prize' 
          :class="cn({ 'border-2 border-red-500 z-[2]': idx == 1, 'opacity-30 border-2 border-black': idx !== 1 })"
        )
      SpinButton.absolute.-bottom-12.inset-x-0(
        :isSpinning = 'isSpinning',
        @start='start'
        @stop='stop'
      ) 

</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";
import { promiseTimeout, until } from "@vueuse/core";
import { cn } from "../utils/tailwind";
import _cloneDeep from "lodash-es/cloneDeep";
import _shuffle from "lodash-es/shuffle";

const emit =  defineEmits<{
  (e: 'fire-animation') : void
  (e: 'update-result', result: number) : void
}>()

// Import components
import SpinItem from "./SpinItem.vue";
import SpinButton from "./SpinButton.vue";
// DEFINE
let timeout;
let interval;
const CONTAINER_HEIGHT = 420;
const VIEW_ITEMS = 3;

// GIFTS
const items = ref<number[]>([]);
const cloneItems = ref<number[]>([]);

// api function to get wheels
async function getItems() {
  await promiseTimeout(200);
  items.value = [
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
  ];
}

async function getTransactions() {
  await promiseTimeout(200);
  return [15, 2, 9, 1, 1];
}
// api function to get result
async function submitWheel() {
  await promiseTimeout(200);
  let result = 15; // response from BE
  return result;
}

// Each time we call the api, the items will be assigned, and we will clone them
// Check if items have changed or not, and clone it
onMounted(async ()=>{
  await getItems()
  await getTransactions()
})
watch(
  () => items,
  () => {
    cloneItems.value = _cloneDeep(items.value);
  },
  { deep: true },
);

// STATE
const isSpinning = ref(false); // is spin loading
const isReceivingResult = ref(false); // is waiting for reward result when calling submit api

// ====== MAIN FUNCTION =====
const spinCount = ref(1000);
const gift = ref(-1);

async function callResult() {
  if (isReceivingResult.value) return;
  isReceivingResult.value = true;
  try {
    spinCount.value--;
    // Binding result
    const res = await submitWheel();
    gift.value = res;
  } catch (error) {
    // Rollback spin count
    spinCount.value++;
    // Throw error
    throw error;
  } finally {
    isReceivingResult.value = false;
  }
}
// - Each time the spin is completed, all rewards must be arranged so that the positions displayed on the UI are random.
// - After that, you need to find the previously won prize (according to the array just suffled) to mark it so the user knows it is a previously received gift.
// - The UI displays 3 elements [0, 1, 2], with index position 1 corresponding to the gift
// RENDER RESULT
async function renderResult() {
  //  Suffle items
  _shuffle(items.value);
  // Step 1: call api get transaction from BE, get first element (Final reward received)
  const histories = await getTransactions();
  const lastGift = histories[0]; // Suppose the last gift received was 15
  // Step 2: Find last gift index
  const lastGiftIndex = items.value.findIndex((i) => i === lastGift);
  //   Step 3: Move the gift to index 1

  // not found -> no handle
  if (lastGiftIndex === -1) return;

  // If index > 1, cut the fisrt element and push it at the end of the array (index -1) times
  // Suppose after suffle the original array becomes [3,7,2,1,15,...] -> lastGiftIndex = 4
  // We need to take the first (4-1 = 3) elements one by one and drop them to the bottom of the array
  if (lastGiftIndex > 1) {
    for (let i = 0; i < lastGiftIndex - 1; i++) {
      items.value.push(items.value.shift());
    }
  }
  // If index < 1, only index is 0, cut the last element, then push it to the beginning of the array 1 time
  // [15,7,2,1,6,..., 3] -> lastGiftIndex = 0
  // Cut 3 and push to beginning of the array -> [3,15,7,...]
  else if (lastGiftIndex < 1) {
    items.value.unshift(items.value.pop());
  }
  // else -> index == 1 -> no handle
}

// START FUNCTION
async function start() {
  if (isSpinning.value) return;
  if (!spinCount.value) {
    // Toast error: 'You do not have enough number of spin'
    return;
  }
  try {
    // Step 1: Stop "victory sound" & Play "spin sound" & spin
    isSpinning.value = true;
    // stopVictorySound()
    // playSpinSound()
    // Step 2: Get result
    await callResult();
    // Step 3: Suffle Origin Wheels & render last result to UI
    await renderResult();
    // Step 4: set time out 5 second to stop
    timeout = setTimeout(() => {
      stop();
    }, 5000);
  } catch (error) {
    throw error;
  }
}
// STOP FUNCTION
async function stop() {
  // If receiving results or !spinning -> not allow stop
  if (!isSpinning.value) return;
  await until(isReceivingResult).toBe(false);
  // Step 1: clear timeout and interval
  clearTimeout(timeout);
  clearInterval(interval);
  // Step 2: set isSpinning to stop
  isSpinning.value = false;
  // Step 3: stop "spin sound" and play "victory sound"
  // stopSpinSound()
  // playVictorySound()
  // Step 4: Fire animation
  emit("fire-animation");
  // Step 5: Emit result to container and show result
  emit("update-result", gift.value);
  console.log(123)
  // Step 6: Call getPrize to update lastest airdrop point
  // getSpinPrize(spinId);
}

// ==== UI Style (calculate container height) ===
const calculatePx = (length: number) => {
  return (CONTAINER_HEIGHT / VIEW_ITEMS) * length;
};

const translateFullPx = computed(
  () => `-${calculatePx(cloneItems.value.length - VIEW_ITEMS)}px`,
);

const containerHeight = `${CONTAINER_HEIGHT}px`;
const itemHeight = `${CONTAINER_HEIGHT / VIEW_ITEMS}px`;
// Animation time
const elStyle = computed(() => {
  const eachItem = 40; // ms
  let time = (eachItem * cloneItems.value.length) / 1000;
  let formattedTime = parseFloat(time.toFixed(2));
  return {
    "--animation-time": `${formattedTime}s`,
  };
});
</script>

<style lang="stylus" scoped>
.spin-view
  max-height: v-bind(containerHeight);

.spin-item
  height: v-bind(itemHeight);


.spinning {
  animation-name spinx
  animation-duration var(--animation-time)
  animation-timing-function linear
  animation-iteration-count infinite
}

@keyframes spinx {
    0% {
        transform: translateY(0);
    }
    100% {
        transform: translateY(v-bind(translateFullPx));
    }
}
</style>
