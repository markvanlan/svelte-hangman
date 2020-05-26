<script>
  import { onMount } from "svelte";

  onMount(resetGame);

  const ALLOWED_MISSES = 5;
  const LOST_STATE = "lost";
  const WON_STATE = "won";
  const PLAYING_STATE = "started";

  let word,
    wordMap,
    started,
    misses,
    guessed,
    progress,
    progressText,
    missText,
    missCount,
    gameState;

  $: invalidWord = !word || word.replace(/ /g, "").length === 0;
  $: imageSrc = word ? `Hangman-${missCount}.png` : "";
  $: missesText = misses ? `Misses: ${missText}` : "";

  function resetGame() {
    word = "";
    started = false;
    wordMap = {};
    misses = [];
    guessed = [];
    progress = [];
    missText = "";
    missCount = 0;
    gameState = PLAYING_STATE;
  }

  function startGame() {
    started = true;
    for (let index = 0; index < word.length; index++) {
      const letter = word[index].toLowerCase();
      if (letter !== " ") {
        wordMap[letter] = wordMap[letter]
          ? wordMap[letter].concat([index])
          : [index];
        progress.push("X");
      } else {
        progress.push(" ");
      }
    }
    progressText = progress.join("");
  }

  function hit(letter) {
    wordMap[letter].forEach((index) => {
      progress[index] = letter;
    });
    delete wordMap[letter];
    progressText = progress.join("");

    if (Object.keys(wordMap).length === 0) {
      gameState = WON_STATE;
    }
  }

  function miss(letter) {
    misses.push(letter);
    missCount += 1;
    missText = misses.join(", ");

    if (missCount > ALLOWED_MISSES) {
      gameState = LOST_STATE;
    }
  }

  function keydown(event) {
    if (
      started &&
      gameState === PLAYING_STATE &&
      event.keyCode < 91 &&
      event.keyCode > 64
    ) {
      const letter = event.key;
      if (guessed.includes(letter)) return;
      guessed.push(letter);

      if (wordMap[letter] === undefined) {
        miss(letter);
      } else {
        hit(letter);
      }
    }
  }
</script>

<style>
  main {
    text-align: center;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }
  h2 {
    font-size: 3em;
  }
  .success {
    color: green;
  }
  .defeat {
    color: red;
  }
  #progress {
    letter-spacing: 3px;
  }
</style>

<svelte:window on:keydown={keydown} />
<main>
  {#if started}
    {#if gameState === PLAYING_STATE}
      <h1>Guess the letters</h1>
    {:else if gameState === WON_STATE}
      <h2 class="success">Congrats, you guessed the word "{word}"</h2>
    {:else if gameState === LOST_STATE}
      <h2 class="defeat">Unfortunately you lost. The word was "{word}"</h2>
    {/if}
    <img src={imageSrc} />
    <p id="progress">{progressText}</p>
    <p>{missesText}</p>
    {#if gameState === WON_STATE || gameState === LOST_STATE}
      <button on:click={resetGame}>Play again</button>
    {/if}
  {:else}
    <h1>Welcome to hangman!</h1>
    <label>
      Player 1, enter a word to be guessed:
      <input type="password" bind:value={word} />
    </label>
    <button disabled={invalidWord} on:click={startGame}>Start game</button>
  {/if}

</main>
