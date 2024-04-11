<script setup lang="ts">
interface Props {
  letterClasses: string[][];
  wordClasses: string[];
  words: string[];
}

const props = defineProps<Props>();
</script>

<template>
  <x-word
    :class="props.wordClasses[indexWord]"
    v-for="(word, indexWord) in props.words"
    :key="indexWord"
  >
    <x-letter
      :class="props.letterClasses[indexWord][indexLetter]"
      v-for="(letter, indexLetter) in word.split('')"
      :key="indexLetter"
    >
      {{ letter }}
    </x-letter>
  </x-word>
</template>

<style scoped>
x-word {
  border-bottom: 2px solid transparent;
  transition: border-color 0.3 ease-in-out;

  &.marked {
    border-color: var(--red);
  }
}

x-letter {
  color: var(--gray);
  position: relative;

  &.active::before {
    content: '|';
    color: var(--yellow);
    font-size: 14px;
    position: absolute;
    left: -60%;
    animation: blink 1s infinite ease-in-out;
  }

  &.active.isLast::before {
    left: 60%;
  }

  &.correct {
    color: var(--green);
  }

  &.incorrect {
    color: var(--red);
  }
}

@keyframes blink {
  0%,
  25% {
    opacity: 1;
  }

  75% {
    opacity: 0;
  }
}
</style>
