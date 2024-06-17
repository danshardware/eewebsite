<script>
	import '../app.css';
</script>
<h1>Dan's EE Math</h1>
<div class="sidebar">
	<a href="/">Home</a>
	<a href="/Thermister">Thermister Calc</a>
	<a href="/Resister">Resister Calc</a>
</div>
<div class="content">
	<slot />
</div>

<style>
	.sidebar {
		margin: 0;
		padding: 0;
		width: 200px;
		height: 100vh;
		position: fixed;
		height: 100%;
		overflow: auto;
	}

	.sidebar a {
		display: block;
		color: var(--secondary-color);
		background-color: var(--sidebar-color);
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

