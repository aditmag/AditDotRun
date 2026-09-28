<nav class="navbar">
	<div class="navbar-left">
		<a href="/" class="brand-link">
			adit.run()
		</a>
	</div>
	<div class="navbar-right">
		<a href="/archive" class="nav-link">random(stuff)</a>
		<a href="/hi" class="nav-link">say(hi)</a>
	</div>
</nav>

<style>
	.navbar {
		width: 100%;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1.5rem 0 1.5rem 0;
		box-sizing: border-box;
		font-family: 'JetBrains Mono', monospace;
		background: white;
		z-index: 10;
		border-bottom: 1px solid #eee;
		position: static;
	}
	.navbar-left {
		font-weight: 700;
		letter-spacing: 0.05em;
		color: #222;
		font-size: 14px;
	}
	.navbar-right {
		display: flex;
		gap: 2rem;
	}
	.nav-link {
		color: #2563eb;
		text-decoration: underline;
		font-size: 14px;
		transition: color 0.2s;
		text-underline-offset: 2px;
	}
	.nav-link:hover {
		color: #1d4ed8;
		text-decoration-thickness: 2px;
	}
	.brand-link {
		color: #222;
		text-decoration: none;
	}
</style>
