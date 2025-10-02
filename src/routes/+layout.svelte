<script lang="ts">
	import favicon from '$lib/assets/11.png';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<title>8k - Ruby Calculator</title>
</svelte:head>

{@render children?.()}
