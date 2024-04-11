<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import TheTimer from './components/TheTimer.vue';
import WordDisplay from './components/WordDisplay.vue';

// Define constants
const INITIAL_TIME = 30;
const TEXT =
  'Este es el texto de prueba mientras agrego mucho texto pero este es el genial typing game.';

// State
const words = ref<string[]>(TEXT.split(' ').slice(0, 32));
const activeWordIndex = ref(0);
const activeLetterIndex = ref(0);

const input = ref('');
const currentWord = computed<string>(() => words.value[activeWordIndex.value]);
const currentLetter = computed<string>(
  () => words.value[activeWordIndex.value][activeLetterIndex.value]
);
const currentMaxLength = computed<number>(() => currentWord.value.length);

const wordClasses = ref<string[]>(['active']);
const letterClasses = ref<string[][]>(
  Array.from({ length: words.value.length }, (_, index) => (index === 0 ? ['active'] : []))
);

//Components
const theTimer = ref<null | InstanceType<typeof TheTimer>>(null);
const inputRef = ref<null | HTMLInputElement>(null);

//User data
// const userInputs = ref<string[]>([]);

const onKeyDown = (event: KeyboardEvent) => {
  const { key } = event;
  if (key === ' ') {
    event.preventDefault();
    //   //Save what the user inputs to in case he skips to the next word without end it all he can comeback where the word was left
    //   userInputs.value[activeWordIndex.value] = input.value;
    input.value = '';
    letterClasses.value[activeWordIndex.value].length = 0;
    const letterClassesLength = letterClasses.value[activeWordIndex.value].length;
    const notAllCorrect = letterClasses.value[activeWordIndex.value].some(
      (letter) => letter === 'incorrect'
    );
    //   //If all letter are not correct
    if (letterClassesLength !== currentMaxLength.value || notAllCorrect) {
      //     //Mark those that the user skip through spacebar with an X
      //     for (let i = letterClassesLength; i < currentMaxLength.value; i++) {
      //       letterClasses.value[activeWordIndex.value][i] = 'x';
      //     }
      wordClasses.value[activeWordIndex.value] = 'marked';
    }
    activeWordIndex.value++;
    return;
  }
  // if (key === 'Backspace') {
  //   if (
  //     input.value.length === 0 &&
  //     activeWordIndex.value > 0 &&
  //     wordClasses.value[activeWordIndex.value - 1] === 'marked'
  //   ) {
  //     event.preventDefault();
  //     activeWordIndex.value--;
  //     input.value = userInputs.value[activeWordIndex.value];
  //     wordClasses.value[activeWordIndex.value] = '';
  //   }
  // }
};

const onKeyUp = () => {
  const inputLength = input.value.length;
  letterClasses.value[activeWordIndex.value].length = 0; //clean up classes
  input.value.split('').forEach((char, index) => {
    const isCorrect = currentWord.value[index] === char ? 'correct' : 'incorrect';
    letterClasses.value[activeWordIndex.value][index] = isCorrect;
    activeLetterIndex.value = index + 1;
  });

  if (inputLength <= currentMaxLength.value - 1) {
    letterClasses.value[activeWordIndex.value][inputLength] = 'active';
  } else {
    letterClasses.value[activeWordIndex.value][inputLength - 1] += ' active isLast';
  }
};

const startTimer = () => {
  theTimer.value?.startTimer(30);
};

onMounted(() => {
  document.addEventListener('keydown', () => {
    inputRef.value?.focus();
  });
});
</script>

<template>
  <main>
    <section id="game">
      <TheTimer ref="theTimer" :duration="INITIAL_TIME" />
      <p>
        <WordDisplay :letterClasses="letterClasses" :wordClasses="wordClasses" :words="words" />
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
</style>
