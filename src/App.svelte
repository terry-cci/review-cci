<script lang="ts">
  import _ from "lodash";
  import oldQuestions from "./data/oldQuestions.json";

  type Choice = {
    text: string;
    correct: boolean;
  };

  type Question = {
    text: string;
    choices: Choice[];
  };

  const questions: Question[] = [];

  const oldQuestionsSplited = oldQuestions.trim().split("\n");

  for (let i = 0; i < oldQuestionsSplited.length; i += 4) {
    const questionText = oldQuestionsSplited[i].trim();

    const choices: Choice[] = [];

    for (let j = 0; j < 3; j++) {
      let choiceText = oldQuestionsSplited[i + j + 1].trim();
      const correct = choiceText.endsWith("*");
      if (correct) choiceText = choiceText.slice(0, -1);

      choices.push({
        text: choiceText,
        correct,
      });
    }

    questions.push({
      text: questionText,
      choices,
    });
  }

  let shuffledQuestions = _.shuffle(questions);

  $: currentQuestion = shuffledQuestions[0];

  let selectedChoiceIdx: number | undefined = undefined;

  $: selectedChoice =
    selectedChoiceIdx !== undefined
      ? currentQuestion.choices[selectedChoiceIdx]
      : undefined;

  function nextQuestion() {
    const currentQuestion = shuffledQuestions.shift();

    if (!selectedChoice?.correct && currentQuestion) {
      shuffledQuestions.push(currentQuestion);
    }

    // trigger re-render
    shuffledQuestions = shuffledQuestions;

    selectedChoiceIdx = undefined;
  }

  $: if (selectedChoiceIdx !== undefined) {
    setTimeout(nextQuestion, 1500);
  }
</script>

<main class="p-4 min-h-[95vh]">
  <p class="mb-6 text-sm">剩餘問題: {shuffledQuestions.length}</p>

  {#if currentQuestion}
    <p class="text-lg">{currentQuestion.text}</p>

    <ul class="mt-4">
      {#each currentQuestion.choices as choice, i}
        <li
          class="py-1 px-2"
          on:click={() =>
            selectedChoiceIdx === undefined && (selectedChoiceIdx = i)}
          class:bg-green-500={selectedChoice && choice.correct}
          class:bg-red-500={selectedChoiceIdx === i && !choice.correct}
          class:text-white={selectedChoiceIdx === i ||
            (selectedChoice && choice.correct)}
          class:font-bold={selectedChoiceIdx === i ||
            (selectedChoice && choice.correct)}
        >
          {choice.text}
        </li>
      {/each}
    </ul>
  {:else}
    <p>你已完成所有題目</p>
  {/if}
</main>

<footer class="px-2 text-center">Made by Terry C.C.I.</footer>
