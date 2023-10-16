<script lang="ts">
  import _ from "lodash";
  import questionsText from "./data/iam-questions";

  type Choice = {
    text: string;
    correct: boolean;
  };

  type Question = {
    id: number;
    text: string;
    choices: Choice[];
  };

  const questions: Question[] = [
    ...questionsText.matchAll(
      /(\d+?)\.? (.+?)A\. (.+?)B\. (.+?)C\. (.+?)D\. (.+?)(E\. (.+?))?答案：([ABCDE])/gs
    ),
  ].map((q) => {
    // console.debug(q);
    const question: Partial<Question> = {};

    question.text = q[2].trim();
    question.id = parseInt(q[1]);

    question.choices = [3, 4, 5, 6, 8]
      .map((i) =>
        q[i]
          ? {
              text: q[i].trim(),
              correct: false,
            }
          : undefined
      )
      .filter((v) => v) as Choice[];

    question.choices[q[9].charCodeAt(0) - "A".charCodeAt(0)].correct = true;

    _.shuffle(question.choices);

    return question as Question;
  });

  // console.debug(questions);

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
    // console.debug(currentQuestion);
  }
</script>

<main class="p-4 min-h-[80vh]">
  <p class="mb-6 text-sm">剩餘問題: {shuffledQuestions.length}</p>

  {#if currentQuestion}
    <p class="text-lg whitespace-pre-line">
      {currentQuestion.id}. {currentQuestion.text}
    </p>

    <ol class="mt-4">
      {#each currentQuestion.choices as choice, i}
        <li
          class="py-3.5 px-2 whitespace-pre-line"
          on:click={() =>
            selectedChoiceIdx === undefined && (selectedChoiceIdx = i)}
          class:bg-green-600={selectedChoice && choice.correct}
          class:bg-red-500={selectedChoiceIdx === i && !choice.correct}
          class:text-white={selectedChoiceIdx === i ||
            (selectedChoice && choice.correct)}
          class:font-bold={selectedChoiceIdx === i ||
            (selectedChoice && choice.correct)}
        >
          {["A", "B", "C", "D", "E"][i]}. {choice.text}
        </li>
      {/each}
    </ol>
  {:else}
    <p>你已完成所有題目</p>
  {/if}
</main>

<footer class="px-2 text-center">Made by Terry C.C.I.</footer>
