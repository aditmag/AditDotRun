<script>
	import Nav from '$lib/Nav.svelte';
</script>

<div class="thin-centered">
	<Nav />
		<main style="margin-top:2.5rem;">
			 <h1>
				ADIT • आदित •
				<span class="dogra-text">
					𑠁𑠛𑠭𑠙
				</span>
			</h1>
			<p>i’m a 20y/o army 
							<span class="tooltip-underline">
								brat
								<span class="tooltip-box">
									<span style="font-weight:700;">brat</span> (abbreviation)<br/>
									<span style="color:#888;">/bræt/</span><br/>
									born, raised, and transferred
								</span>
							</span>, currently studying at <a href="https://youtu.be/MShbP3OpASA?si=2S4O7T4Cwfd9VN0g&t=2997" target="_blank">aalto</a> in finland, who grew up attending 9 schools in 7 cities across india before graduating from <a href="https://youtu.be/8w8oGfe7TbE" target="_blank">sanskriti school</a>, new delhi.
						</p>
						<br/>
						<p>most recently, i co-founded <a href="https://nullfellows.com" target="_blank">null fellows</a> and joined <a href="https://ouraring.com" target="_blank">oura</a> as the founding member of a 3-person innovation team.</p>
						<!-- <br> -->

                        <br/><br/>

                        <p><b>STUFF I'VE DONE</b></p>
                        <ul>
							<li>- 400k views + nvidia prize — treehacks @ stanford. </li>
							<li>- 1/6 builders handpicked by marten mickos (mysql).</li>
                            <li>- neurips workshop + natl sci fair at 17.</li>
                            <li>- slipped on a banana peel on the road like a cartoon.</li>
                        </ul>

                        <br/><br/>

                        <p><b>PREVIOUS WORK</b></p>
                        <ul>
							<li>- <a href="https://paebbl.com" target="_blank">paebbl.com</a>: turned co<sub>2</sub> in the air to bricks.</li>
							<li>- <a href="https://aaltoes.com" target="_blank">aaltoes.com</a>: made people build for the love of the game.</li>
                            <li>- <a href="https://smu.edu.sg" target="_blank">smu.edu.sg</a>: probabilistic early rumour detection models</li>
                            <li>- <a href="https://web.archive.org/web/20240727225148/https://projectbeta.club/" target="_blank">projectbeta.club</a>: became president to skip classes.</li>
                            <li>- <a href="https://web.archive.org/web/20220125185014/https://slingshotahead.com/" target="_blank">slingshotahead.com</a>: future 'lebrons of tech'.</li>  
                        </ul>

						<br/><br/>

						i hope to eventually make hyperrealistic robot dinosaurs and open a real jurassic park.

                    </main>
</div>

<style>

	a {
		color: #2563eb;
		text-decoration: underline;
		text-underline-offset: 2px;
		transition: color 0.2s;
	}
	a:hover {
		color: #1d4ed8;
		text-decoration-thickness: 2px;
	}
			.thin-centered {
				font-family: 'JetBrains Mono', monospace;
				font-size: 14px;
				width: 40vw;
				min-width: 320px;
				max-width: 600px;
				margin: 1.5rem auto 3rem auto;
				padding: 2rem 2vw;
				background: transparent;
				box-sizing: border-box;
			}
	:global(html), :global(body) {
		font-family: 'JetBrains Mono', monospace;
		font-size: 14px;
		margin: 0;
		padding: 0;
		background: #fff;
	}
	h1 {
		font-size: 20px;
		font-weight: 700;
		margin: 1.5rem 0 1rem 0;
		font-family: 'JetBrains Mono', monospace;
	}
		.tooltip-underline {
			position: relative;
			display: inline-block;
			border-bottom: 1px dashed #888;
			cursor: pointer;
			transition: border-color 0.2s, color 0.2s;
			color: #888;
		}
		.tooltip-underline:hover {
			border-bottom: 1px solid #222;
			color: #222;
		}
	.tooltip-underline:hover {
		border-bottom: 1px solid #222;
	}
	.tooltip-box {
		display: none;
		position: absolute;
		left: 50%;
		top: 120%;
		transform: translateX(-50%);
		min-width: 210px;
		background: #fff;
		color: #222;
		border: 1px solid #e0e0e0;
		border-radius: 8px;
		box-shadow: 0 2px 12px 0 rgba(0,0,0,0.07);
		padding: 0.75rem 1rem;
		font-size: 13px;
		font-family: 'JetBrains Mono', monospace;
		z-index: 100;
		white-space: pre-line;
		pointer-events: none;
	}
	.tooltip-underline:hover .tooltip-box {
		display: block;
		animation: fadeIn 0.18s;
	}
	@keyframes fadeIn {
		from { opacity: 0; transform: translateX(-50%) translateY(10px); }
		to { opacity: 1; transform: translateX(-50%) translateY(0); }
	}
	.dogra-text {
		font-family: 'Noto Serif Dogra', serif;
		font-size: 20px;
		vertical-align: middle;
		margin-left: 0.5em;
		letter-spacing: normal;
		font-weight: 700;
	}
</style>
