<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';

interface Props {
  duration: number;
}

interface Emits {
  (e: 'gameOver'): void;
}

const props = defineProps<Props>();
const emits = defineEmits<Emits>();

const timer = ref(0);
let intervalId: undefined | number;

const startTimer = (time: number) => {
  if (intervalId !== undefined) {
    clearInterval(intervalId)
  }
  timer.value = time
  intervalId = setInterval(countDown, 1000);
};

const countDown = () => {
  timer.value--;

  if (timer.value === 0) {
    clearInterval(intervalId);
    emits('gameOver');
  }
};

onMounted(() => {
  startTimer(props.duration);
});

onUnmounted(() => {
  clearInterval(intervalId)
})

defineExpose({
  timer,
  startTimer
})
</script>

<template>
  <time>{{ timer }}</time>
</template>

<style scoped>
time {
  color: var(--yellow);
}
</style>
