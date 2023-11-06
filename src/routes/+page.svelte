<script>
	import { fade } from "svelte/transition";


	import Number from "../lib/components/Number.svelte";
	import Tile from "../lib/components/Tile.svelte";

	let Position = [
		0, 0, 0, 0,
		0, 0, 0, 0,
		0, 0, 0, 0,
		0, 0, 0, 0
	];

	let free_indices = [];
	let win = false;
	let death_countdown = [];
	let endgame = false;
	let realized_movement = false;
	let score = 0;
	let max_score = 0;


	const generate_numbers = () => {
		let position_in_free_tile_list = Math.floor(Math.random() * free_indices.length)
		Position[free_indices[position_in_free_tile_list]] = 2 + Math.round(Math.random(), 0) * 2
	} 

	const find_free_tiles = () => {

		free_indices = []

		for (let i = 0; i < Position.length; i++) {

			if (Position[i] === 0) {
				free_indices.push(i)
				continue;
			}

			if (Position[i] === 2048) {
				win = true;
			}


		}

	}

	const update_score = () => {

		score = 0;

		for (let i = 0; i < Position.length; i++) {

			score += Position[i]

			if (score > max_score) {
				max_score = score;
			}

		}

	}

	const init_game = () => {
		find_free_tiles()
		generate_numbers()
		find_free_tiles()
		generate_numbers()
		update_score()
	}

	init_game()


	const serialize = (currentID, empty_index, prev_empty = false) => {

		if (currentID === empty_index && Position[currentID] === 0) false;

		if (Position[currentID] !== 0) {
			if (currentID !== empty_index) {
				Position[empty_index] = Position[currentID]
				Position[currentID] = 0
				realized_movement = true;
			}
			if (prev_empty !== false && Position[prev_empty] === Position[empty_index]) {

				Position[prev_empty] = Position[empty_index] + Position[prev_empty]
				Position[empty_index] = 0
				realized_movement = true;

				return false
			}

			return true
		}

		return false
	}

	const left = () => {
		// Serialize left:

		// Rows:
		for (let i = 0; i < 4; i++) {
			// Keeping track of row's empty index
			let empty_index = i * 4;
			let prev_empty = false

			for (let j = 0; j < 4; j++) {

				let currentID = 4 * i + j

				if (serialize(currentID, empty_index, prev_empty)) {
					prev_empty = empty_index
					empty_index++
				}

			}

		}
	}

	const right = () => {

		// Rows:
		for (let i = 0; i < 4; i++) {
			// Keeping track of row's empty index
			let empty_index = 3 + i * 4;
			let prev_empty = false

			for (let j = 0; j < 4; j++) {

				let currentID = 4 * (i + 1) - 1 - j
				if (serialize(currentID, empty_index, prev_empty)) {
					prev_empty = empty_index
					empty_index--
				}

			}
		}
	}

	const up = () => {

		// Columns:
		for (let i = 0; i < 4; i++) {
			// Keeping track of row's empty index
			let empty_index = i;
			let prev_empty = false;

			// Rows
			for (let j = 0; j < 4; j++) {
				
				let currentID = i + 4 *j
				if (serialize(currentID, empty_index, prev_empty)) {
					prev_empty = empty_index
					empty_index += 4
				}
			}

		}
	}
	
	const down = () => {

		// Rows:
		for (let i = 0; i < 4; i++) {
			// Keeping track of row's empty index
			let empty_index = i + 12;
			let prev_empty = false

			for (let j = 0; j < 4; j++) {

				let currentID = (12 + i) - 4 * j
				if (serialize(currentID, empty_index,prev_empty)) {
					prev_empty = empty_index
					empty_index -= 4
				}

			}
		}

	}

	const restart = () => {
		Position = [
			0, 0, 0, 0,
			0, 0, 0, 0,
			0, 0, 0, 0,
			0, 0, 0, 0
		]
		death_countdown = []
		endgame = false
		win = false

		init_game()

	}

	const options = {
		"ArrowLeft": left,
		"ArrowRight": right,
		"ArrowUp": up,
		"ArrowDown": down,
		"Escape": restart

	}

	const control = event => {

		try {
			realized_movement = false;
			options[event.code]()

			if (event.code === "Escape") return;

		} catch (TypeError) {
			console.log("Invalid key. Skipping...")
			console.log(event.code)
		}

		find_free_tiles()

		// Adding a new random number

		if (free_indices.length === 0) {
			if (!death_countdown.includes(event.code)) {
				death_countdown.push(event.code)
			}
		} else {
			death_countdown = []
		}

		if (death_countdown.length === 4) {
			endgame = true
		}

		if (realized_movement) {
			generate_numbers();
		}

		update_score()

	}

</script>

<svelte:window on:keydown={control} />
<main class="field">

	<aside class="score">
		<p>Score: {score}</p>
		<p>Best score in session: {max_score}</p>
	</aside>

	{#each Array(16).keys() as id}
		<Tile {id}> 
			{#if Position[id] != 0}
				<Number value = {Position[id]} />
			{/if}
		</Tile>
	{/each}
	{#if endgame}
		<div class="endgame" transition:fade>
			<p>You Lost.</p>
			<p>Score: {score}</p>
		</div>
	{/if}

	{#if win}
		<div class="endgame" transition:fade>
			<p>You Won.</p>
			<p>Score: {score}</p>
		</div>
	{/if}

</main>


<style lang="scss">

	.field {

		aspect-ratio: 1/1;
		min-width: 100px;

		background-color: gray;
		padding: 4px;
		border-radius: 4px;
		border: 4px solid darkgrey;

		display: grid;
		grid-template-columns: repeat(4, auto);
		gap: 3px;

	}

	.endgame {
		width: 100%;
		height: 100vh;
		z-index: 10;
		background-color: rgba(0, 0, 0, 0.9);
		position: absolute;
		top: 0;left: 0;
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		gap: 4px;
	}

	.score {

		position: absolute;
		top: 0;
		right: 10px;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: left;
		padding: 20px 10px;	
		gap: 4px;

	}

</style>