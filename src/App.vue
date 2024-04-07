<script setup lang="ts">
import { ref, onBeforeMount, computed, onMounted } from 'vue';

// Define constants
const initialTime = 30;
const TEXT =
  'Este es el texto de prueba mientras agrego mucho texto pero este es el genial typing game.';

// State
const inputRef = ref<null | HTMLInputElement>(null);
const time = ref(initialTime);
const words = ref<string[]>([]);
const activeWordIndex = ref(0);
const activeLetterIndex = ref(0);

const input = ref('');
const currentWord = computed<string>(() => words.value[activeWordIndex.value]);
const currentMaxLength = computed<number>(() => currentWord.value.length);

const wordClasses = ref<string[]>([]);
const letterClasses = ref<string[][]>([[]]);

//User data
const userInputs = ref<string[]>([]);

// Initialization
const initGame = () => {
  input.value = '';
  words.value = TEXT.split(' ').slice(0, 32);
  time.value = initialTime;
  activeWordIndex.value = 0;
  activeLetterIndex.value = 0;
  startTimer();
};

const onKeyDown = (event: KeyboardEvent) => {
  const { key } = event;
  letterClasses.value[activeWordIndex.value + 1] = [];
  if (key === ' ') {
    event.preventDefault();
    //Save what the user inputs to in case he skips to the next word without end it all he can comeback where the word was left
    userInputs.value[activeWordIndex.value] = input.value;
    input.value = '';

    const letterClassesLength = letterClasses.value[activeWordIndex.value].length;
    const notAllCorrect = letterClasses.value[activeWordIndex.value].some(
      (letter) => letter === 'incorrect'
    );

    //If all letter are not correct
    if (letterClassesLength !== currentMaxLength.value || notAllCorrect) {
      //Mark those that the user skip through spacebar with an X
      for (let i = letterClassesLength; i < currentMaxLength.value; i++) {
        letterClasses.value[activeWordIndex.value][i] = 'x';
      }
      wordClasses.value[activeWordIndex.value] = 'marked';
    }

    activeWordIndex.value++;
    return;
  }

  if (key === 'Backspace') {
    if (
      input.value.length === 0 &&
      activeWordIndex.value > 0 &&
      wordClasses.value[activeWordIndex.value - 1] === 'marked'
    ) {
      event.preventDefault();
      activeWordIndex.value--;
      input.value = userInputs.value[activeWordIndex.value];
      wordClasses.value[activeWordIndex.value] = '';
    }
  }
};

const onKeyUp = () => {
  if (input.value.length === 0) activeLetterIndex.value = 0;

  letterClasses.value[activeWordIndex.value].length = 0;
  input.value.split('').forEach((char, index) => {
    const isCorrect = currentWord.value[index] === char ? 'correct' : 'incorrect';
    letterClasses.value[activeWordIndex.value][index] = isCorrect;
    activeLetterIndex.value = index + 1;
  });
};

const startTimer = () => {
  const intervalId = setInterval(() => {
    time.value--;

    if (time.value === 0) {
      clearInterval(intervalId);
      gameOver();
    }
  }, 1000);
};

const gameOver = () => {};

onMounted(() => {
  document.addEventListener('keydown', () => {
    inputRef.value?.focus();
  });
});
onBeforeMount(() => initGame());
</script>

<template>
  <main>
    <section id="game">
      <time>{{ time }}</time>
      <p>
        <x-word
          v-for="(word, indexWord) in words"
          :class="[wordClasses[indexWord], { active: indexWord === activeWordIndex }]"
          :key="indexWord"
        >
          <x-letter
            v-for="(letter, indexLetter) in word.split('')"
            :class="[
              letterClasses[indexWord]?.[indexLetter],
              {
                active: indexWord === activeWordIndex && indexLetter === activeLetterIndex,
                'active isLast':
                  indexWord === activeWordIndex &&
                  indexLetter === activeLetterIndex - 1 &&
                  input.length === currentMaxLength
              }
            ]"
            :key="indexLetter"
            >{{ letter }}</x-letter
          >
        </x-word>
      </p>
      <input
        ref="inputRef"
        autofocus
        v-model="input"
        :maxlength="currentMaxLength"
        @keydown="onKeyDown"
        @keyup="onKeyUp"
      />
    </section>
  </main>
</template>

<style scoped>
#section {
  padding: 16px;
  display: flex;
  flex-direction: column;
  max-width: 500px;
}

time {
  color: var(--yellow);
}

input {
  z-index: -999;
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
  opacity: 0;
}

p {
  display: flex;
  flex-wrap: wrap;
  gap: 3px 6px;
  margin: 0;
}

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
