<script>
	import Thermister from '../lib/Thermister.svelte';
</script>

<h1>Dan's EE Math</h1>
<div class="sidebar">
	<a href="/">Home</a>
	<a href="/+page">Thermister Calc</a>
</div>
<div class="content">
	<Thermister />
</div>

<style>
	.sidebar {
		margin: 0;
		padding: 0;
		width: 200px;
		position: fixed;
		height: 100%;
		overflow: auto;
	}

	.sidebar a {
		display: block;
		color: var(--secondary-color);
		padding: 16px;
		text-decoration: none;
	}

	.sidebar a.active {
		background-color: #04aa6d;
		color: var(--secondary-color-dark);
	}

	.sidebar a:hover:not(.active) {
		background-color: var(--secondary-color-dark);
		color: var(--background-color);
	}
	div.content {
		margin-left: 200px;
		padding: 1px 16px;
		height: 1000px;
	}
	@media screen and (max-width: 700px) {
		.sidebar {
			width: 100%;
			height: auto;
			position: relative;
		}
		.sidebar a {
			float: left;
		}
		div.content {
			margin-left: 0;
		}
	}
</style>
