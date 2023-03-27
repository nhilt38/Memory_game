<script>
  import {onMount} from 'svelte'
	import Fireworks from '~icons/noto-v1/fireworks'
	import Cry from '~icons/tabler/mood-cry'
	const icons = [
      'https://api.iconify.design/fluent-emoji-flat:fish.svg?color=%23888888',
      'https://api.iconify.design/emojione-v1:octopus.svg?color=%23888888',
      'https://api.iconify.design/emojione:crab.svg?color=%23888888',
      'https://api.iconify.design/noto:shrimp.svg?color=%23888888',
	]
  
  const PIECE_TO_COVER = 3

  const GAME_LOAD = 0
  const GAME_START = 1
  const GAME_GUESS_BOARD = 2
  const GAME_WIN = 3
  const GAME_LOSE = 4

  let board = []
  let gamePhase = GAME_LOAD
  let gameStartedAt = new Date()
  let gameTime = ""
  let gameTimer = 0
  let isAnsweringFor = -1

  let answerDialog;
  let resultDialog;

  function newBoard() {
    board = Array
      .from(Array(16).keys())
      .map(x => Math.random())
      .map(x => Math.floor(x * icons.length))
      .map(x => ({
          id: x,
          icon: icons[x],
          guessed: x,
          hidden: false,
      }));
  }

  function coverBoard() {
    const toCover = Array
      .from(Array(16).keys())
      .sort(() => 0.5 - Math.random())
      .slice(0,PIECE_TO_COVER)
    for(const i of toCover) {
      board[i].hidden = true
      board[i].guessed = -1
    }
  }

  function openMultiChoice(i) {
    isAnsweringFor = i
    answerDialog.showModal()
  }

  function answer(x) {
    board[isAnsweringFor].guessed = x
    isAnsweringFor = -1
    answerDialog.close()
  }

  function checkAnswerCorrect() {
    for(const cell of board) {
      if(!cell.hidden) continue
      if(cell.guessed != cell.id) return false
    }
    return true
  }

  function restart() {
    newBoard()
    resultDialog.close()
    clearInterval(gameTimer)
    gamePhase = GAME_START
  }

  function updateGameTimer() {
    const formatter = new Intl.DateTimeFormat('en', {
      minute: '2-digit',
      second: '2-digit',
    })
    gameTime = formatter.format(new Date() - gameStartedAt)
  }

  function nextPhase() {
    switch(gamePhase) {
      case GAME_START:
        coverBoard()
        gamePhase = GAME_GUESS_BOARD
        gameStartedAt = new Date()
        gameTimer = setInterval(updateGameTimer)
        break;
      case GAME_GUESS_BOARD:
        if(checkAnswerCorrect()) {
          gamePhase = GAME_WIN
        } else {
          gamePhase = GAME_LOSE
        }
        resultDialog.showModal()
        clearInterval(gameTimer)
        break;
      case GAME_WIN:
      case GAME_LOSE:
        newBoard()
        resultDialog.close()
        gamePhase = GAME_START
        break;
    }
  }
  onMount(() => {
    newBoard()
    gamePhase = GAME_START
  })
</script>

<svelte:head>
  <title>Memory game</title>
</svelte:head>

<header class="flex justify-between md:m-16 m-10">
		<div class="font-bold text-2xl bold">Memory</div>
		<div class="md:w-40 w-36 h-12 bg-slate-300 p-3 font-semibold rounded-lg md:mr-6 mr-0 flex justify-between">
			<span class="text-slate-500">Time</span>
			<span>{gameTime}</span>
		</div>
</header>

<main id="layout" class="w-full">
  <div class="flex justify-center">
    <div class="grid grid-rows-4 grid-cols-4 md:gap-8 gap-4">
      {#each board as cell, index}
        {#if cell.hidden}
          <button on:click={() => openMultiChoice(index)} class="md:w-20 md:h-20 w-16 h-16 rounded-full bg-orange-200 grid place-items-center">
            {#if cell.guessed != -1}
              <img src="{icons[cell.guessed]}" alt="icon" class="aspect-square md:w-10 w-8">
            {/if}
          </button>
        {:else}
          <button disabled class="md:w-20 md:h-20 w-16 h-16 rounded-full bg-slate-200 grid place-items-center">
              <img src="{cell.icon}" alt="icon" class="aspect-square md:w-10 w-8">
          </button>
        {/if}
      {/each}
    </div>
  </div>
</main>

<footer class="flex justify-center md:m-16 m-8">
	<div>
		<button on:click={restart} class="bg-yellow-500 md:mr-12 mr-6 md:w-36 w-28 h-12 rounded-3xl text-white font-semibold hover:bg-amber-400">
      Restart
    </button>
		<button on:click={nextPhase} class="md:ml-12 ml-6 bg-yellow-500 md:w-32 w-28 h-12 text-white font-semibold rounded-3xl hover:bg-amber-400">
      {gamePhase == GAME_START?"Start":"Next"}
    </button>
	</div>
</footer>

<dialog bind:this={answerDialog} 
  on:click|self={() => answerDialog.close()}
  class="md:w-44 w-80 transition ease-in-out delay-150 rounded-lg bg-yellow-400">
  <h3 class="text-white text-center text-xl md:p-10 font-semibold p-4">Please choose your selection</h3>
  <div class="pb-8 flex gap-2">
    {#each icons as icon, index}
    <button on:click={() => answer(index)} class="md:w-20 aspect-square w-16 mx-2 rounded-full bg-slate-200 transition-all hover:bg-yellow-600 grid place-items-center">
      <img src="{icon}" alt="icon" class="md:w-10 md:h-10 w-8 h-8">
    </button>
    {/each}
  </div>
</dialog>

<dialog bind:this={resultDialog} 
  on:click|self={() => resultDialog.close()}
  class="rounded-lg bg-yellow-400 p-10">
  <div class="text-center">
    <h2 class="text-white font-bold text-4xl p-4">Your result</h2>
    <p class="font-semibold text-2xl text-orange-800 p-4">
    {#if gamePhase == GAME_WIN}
      Congratulations
      <Fireworks />
    {:else}
      Better luck next time
      <Cry />
    {/if}
    </p>
    <span class="font-semibold text-xl text-white p-4">Time: {gameTime}</span>
  </div>
</dialog>
