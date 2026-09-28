<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/hicon.jpg';
	import { injectAnalytics } from '@vercel/analytics/sveltekit'

	injectAnalytics();
	
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
	<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Crimson+Text:ital,wght@0,400;0,600;0,700;1,400;1,600;1,700&family=Noto+Serif+Dogra&display=swap" rel="stylesheet">
</svelte:head>

{@render children?.()}
