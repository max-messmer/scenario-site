<script lang="ts">
	let { children } = $props();
</script>

<svelte:head>
	<title>SCENARIO. — Hardcore / Screamo</title>
	<meta name="description" content="SCENARIO. — Hardcore screamo from the depths." />
</svelte:head>

{@render children()}

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(html) {
		scroll-behavior: smooth;
		scrollbar-width: thin;
		scrollbar-color: #69140e #00171f;
	}

	:global(body) {
		background: #00171f;
		color: #cfccd6;
		font-family: 'Barlow Condensed', 'Arial Narrow', sans-serif;
		overflow-x: hidden;
	}

	@media (min-width: 601px) {
		:global(body) { cursor: none; }
		:global(a) { cursor: none; }
	}

	:global(::selection) {
		background: #69140e;
		color: #cfccd6;
	}

	:global(::-webkit-scrollbar) {
		width: 6px;
	}

	:global(::-webkit-scrollbar-track) {
		background: #00171f;
	}

	:global(::-webkit-scrollbar-thumb) {
		background: #69140e;
		border-radius: 3px;
	}
</style>
