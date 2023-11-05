<script>

	export let id;

</script>

<div class="tile" {id}>
	<slot />
</div>


<style lang="scss">

	.tile {

		width: 100px;
		height: 100px;
		background-color: #5599bb;
		padding: 4px;
		border: 4px solid black;
		border-radius: 4px;
		display: flex;
		justify-content: center;
		align-items: center;

	}


</style>