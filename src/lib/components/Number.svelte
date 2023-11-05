<script>

	import { slide } from "svelte/transition";

	export let value;

	const scale = {
		2: '#eee4da',
		4: '#ede0c8',
		8: '#f2b179',
		16: '#f59563',
		32: '#f67c5f',
		64: '#f65e3b',
		128: '#edcf72',
		256: '#edcc61',
		512: '#edc850',
		1024: '#edc53f',
		2048: '#edc22e'
	}

</script>

<div class="number" style="background-color: {scale[value]}" transition:slide>
	<p>{value}</p>
</div>

<style lang='scss'>

	.number {

		background-color: black;
		border-radius: 4px;
		width: 90%;
		height: 90%;

		display: flex;
		justify-content: center;
		align-items: center;

		transition: background-color .3s ease-in-out;

	}

	p {

		font-size: 1.1rem;

	}

</style>