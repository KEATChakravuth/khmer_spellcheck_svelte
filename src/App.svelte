<script>
	import Document from "./components/Document.svelte";
	import Suggestion from "./components/Suggestion.svelte";
</script>

<main>
	<div class="container">
		<Document/>
		<Suggestion/>
	</div>
</main>

<style>
	@media print {
		@page {
		size: A4;
		margin: 1cm;
		}
	}
	.container{
		width: 100%;
		display: flex;
		justify-content: center;
		gap: 1rem;
		padding: 3rem;
	}
</style>