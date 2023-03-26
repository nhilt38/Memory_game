<script>
	import Fireworks from '~icons/noto-v1/fireworks'
	import Cry from '~icons/tabler/mood-cry'
	const icons = [
		'https://api.iconify.design/fluent-emoji-flat:fish.svg?color=%23888888',
		'https://api.iconify.design/emojione-v1:octopus.svg?color=%23888888',
		'https://api.iconify.design/emojione:crab.svg?color=%23888888',
		'https://api.iconify.design/noto:shrimp.svg?color=%23888888',
	]
//Shuffle icons
const shuffle = (icons) => {
		const clonedArray = [...icons]
		for (let index = clonedArray.length - 1; index > 0; index--) {
			const randomIndex = Math.floor(Math.random() * (index + 1))
			const original = clonedArray[index]
			clonedArray[index] = clonedArray[randomIndex]
			clonedArray[randomIndex] = original
		}
		return clonedArray
}
const items = shuffle([...icons, ...icons, ...icons, ...icons])	
//Initial values
let showResult = false,
	nextStep = false,
	openSelection = false,
	winDialog = false,
	loseDialog = false
//Switch button next to submit
function toggle() {
	nextStep = true
}
//Format time
const formatter = new Intl.DateTimeFormat('en', {
		minute: '2-digit',
		second: '2-digit',
	})
//Game state
const state ={
	gameStart: false,
	time: 0,
	loop: 0,
}
//Start game
var imgList
const start = () => {
	state.gameStart = true
	state.loop = setInterval(() => {
		state.time++
	})
	imgList = document.querySelectorAll('.img')
	var numbers = [];
			for (let i = 0; i < 4; i++) {
			var random =  Math.floor(Math.random() * 16)
			var check = numbers.includes(random)
			if(check === false) {
				numbers.push(random);
				imgList[random].classList.add('hide')
			} else {
			while(check === true){
				random = Math.floor(Math.random() * 16)
				check = numbers.includes(random)
				if(check === false){
					numbers.push(random)
					imgList[random].classList.add('hide')
				}
				}
			}
			}
}
//Restart
const restart = () =>{
	state.gameStart = false
	state.time = 0
	clearInterval(state.loop)
	state.loop = 0
	nextStep = false
	showResult = false
	imgList = document.querySelectorAll('.img')
	imgList.forEach((imgEl) => {
		imgEl.classList.remove('hide')
		imgEl.parentElement.classList.remove('selected')
	})
}
//Submit
const submit = () => {
	showResult = true
	clearInterval(state.loop)
	state.loop = 0
}
//Open selection
let currentScr =['']
function handleSelect (event) {	
	if(event.target.classList.contains('hide')){
		openSelection = true
		event.target.parentElement.classList.add('selected')
		currentScr = event.target.getAttribute('src')
	}
	else{
		console.log('error')
	}
}
let resultContainer = ['']
function checkMatch (event) {
		if(event.target.getAttribute('src') !== currentScr ){
				resultContainer.push('false')
			}
		else{
				resultContainer.push('true')		
			} 				
}
function showResults (){
	if(resultContainer.includes('false') || resultContainer.length < 4){
		loseDialog = true,
		winDialog = false
	}
	else {
		winDialog = true,
		loseDialog = false
	}
}
</script>
	<svelte:head>
		<title>Memory game</title>
		<meta name="description" content="Svelte demo app" />
	</svelte:head>
<header class="flex justify-between m-16 max-[767px]:m-10">
		<div class="font-bold text-2xl bold">Memory</div>
		<div class="w-40 max-[767px]:w-36 h-12 bg-slate-300 p-3 font-semibold rounded-lg mr-6 max-[767px]:mr-0 flex justify-between">
			<span class="text-slate-500">Time</span>
			<span>{formatter.format(state.time)}</span>
		</div>
</header>
	<main id="layout" class="w-full">
		<div class="flex justify-center">
			<div class="grid grid-rows-4 grid-cols-4 gap-8 max-[767px]:gap-4">
				{#each items as item, index}
				<button on:click={handleSelect} data-id="{index}" class="btn w-20 h-20 max-[767px]:w-16 max-[767px]:h-16 rounded-full bg-slate-200 relative">				
					<img src="{item}" alt="icon" class="img w-10 h-10 max-[767px]:w-8 max-[767px]:h-8">
				</button>
				{/each}
			</div>
		</div>
	</main>
<!-- round 2-->
<div class:selection={openSelection} class="transition ease-in-out delay-150 hidden rounded-lg bg-yellow-400 max-[767px]:w-80">
	<h3 class="text-white text-center text-xl p-10 font-semibold max-[767px]:p-4">Please choose your selection</h3>
	<div class="pb-8">
		{#each icons as icon}
		<button on:click={() => openSelection = false} on:click={checkMatch} class="w-20 h-20 max-[767px]:w-16 max-[767px]:h-16 max-[767px]:mx-2 mx-4 text-center rounded-full bg-slate-200 relative hover:transition-all hover:bg-yellow-600">
			<img src="{icon}" alt="icon" class="text-center w-10 h-10 max-[767px]:w-8 max-[767px]:h-8">
		</button>
		{/each}
	</div>
</div>
<footer class="flex justify-center m-16 max-[767px]:m-8">
	<div>
		<button on:click={restart} class="bg-yellow-500 mr-12 max-[767px]:mr-6 w-36 max-[767px]:w-28 h-12 rounded-3xl text-white font-semibold hover:bg-amber-400">Restart</button>
		{#if nextStep}
		<button on:click|once={toggle} on:click={submit} on:click={showResults} class="ml-12 max-[767px]:ml-6 bg-yellow-500 w-32 max-[767px]:w-28 h-12 text-white font-semibold rounded-3xl hover:bg-amber-400">Submit</button>
		{/if}
		{#if !nextStep}
		<button on:click={start} on:click|once={toggle} class="ml-12 max-[767px]:ml-6 bg-yellow-500 w-32 max-[767px]:w-28 h-12 text-white font-semibold rounded-3xl hover:bg-amber-400">Start</button>
		{/if}
	</div>
</footer>
<!--Result-->
<div class:dialog={showResult} class="hidden rounded-lg bg-yellow-400 max-[767px]:w-80 max-[767px]:h-2/6">
	<div class="text-center">
		<h2 class="text-white font-bold text-4xl p-4">Your result</h2>
		<p class:show={winDialog} class="hidden font-semibold text-2xl text-orange-800 p-4">
			Congratulations
			<Fireworks />
			</p>
		<p class:show={loseDialog} class="hidden font-semibold text-2xl text-orange-800 p-4">
			Better luck next time
			<Cry />
		</p>
		<span class="font-semibold text-xl text-white p-4">Time: {formatter.format(state.time)}</span>
	</div>
</div>