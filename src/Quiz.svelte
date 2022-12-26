<script>
    import { fly } from "svelte/transition";
    import Question from "./Question.svelte";
    import Modal from "./Modal.svelte";
    import { score } from "./store";

    let quiz = getQuiz();
    let activeQuestion = 0;
    let isModalOpen = false;

    async function getQuiz() {
        const res = await fetch(
            "https://opentdb.com/api.php?amount=10&category=18&type=multiple"
        );
        const quiz = await res.json();
        return quiz;
    }

    function nextQuestion() {
        activeQuestion += 1;
    }

    function resetQuiz() {
        score.set(0);
        quiz = getQuiz();
        activeQuestion = 0;
        isModalOpen = false;
    }

    $: if ($score > 7) {
        isModalOpen = true;
    }

    $: questionNumber = activeQuestion + 1;
</script>

<div>
    <button on:click|once={resetQuiz}>New Quiz</button>

    <h3>My Score: {$score}</h3>
    <h4>Question #{questionNumber}</h4>

    {#await quiz}
        Loading...
    {:then data}
        {#each data.results as question, index}
            {#if index === activeQuestion}
                <div
                    in:fly={{ x: 100 }}
                    out:fly={{ x: -200 }}
                    class="fade-wrapper"
                >
                    <Question {nextQuestion} {question} />
                </div>
            {/if}
        {/each}
    {/await}
</div>

{#if isModalOpen}
    <Modal on:close={resetQuiz}>
        <h2>You won!</h2>
        <p>Congratulations!</p>
        <button on:click={resetQuiz}>Start over</button>
    </Modal>
{/if}

<style>
    .fade-wrapper {
        position: absolute;
    }
</style>
