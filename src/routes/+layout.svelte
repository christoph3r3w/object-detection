<script>
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<main>
	<section>
		{@render children()}
	</section>
</main>

<style>

	main {
		position: relative;
		display: grid;
		place-content: center;
		padding: 1em;
		max-width: 100vw;
		height: 100svh;
		margin: 0 auto;
	}
	section {
		max-width: 80dvw;
		width: 100%;
		height: fit-content;
		display: flex;
		flex-direction: column;
		align-items: center;
		overflow:auto;
	}
</style>
