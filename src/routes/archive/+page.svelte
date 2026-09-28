<script>
	import Nav from '$lib/Nav.svelte';
  /*
  import { onMount } from "svelte";

  let song = null;
  let loading = true;
  let error = null;

  onMount(async () => {
    try {
      const res = await fetch(
        "https://lastfm-last-played.biancarosa.com.br/The_AlphaLaser/latest-song"
      );
      if (!res.ok) throw new Error("Failed to fetch song");

      song = await res.json();
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  });
  */
</script>

<!--
idk a potential list of things i like??? porbs shouldnt include music 
coz then ts would fet WAY too long (title of your sex tape).




-->

<div class="thin-centered">
	<Nav />

    <!-- <center>
		⚒️ under construction ⚒️
    </center> -->
	<br>ME, MYSELF, AND I: <br>
	<a href="https://substack.com/@aditmag" target="_blank">substack</a>, <a href="https://letterboxd.com/aditmag" target="_blank">letterboxd</a>, <a href="https://www.last.fm/user/The_AlphaLaser" target="_blank">last.fm</a><br>
	<br>
	CURATED LINKS: <br>
	<a href="http://www.incompleteideas.net/IncIdeas/BitterLesson.html" target="_blank">the bitter lesson</a><br>
	<a href="https://xkcd.com/1211/" target="_blank">birds and dinosaurs - xkcd</a><br>
	<a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgoqQ1qAYT0OQ6j7ezWu8GY2JlpsrW7aBqIwzk98O8snjfF6eiUQGWPKdVQPQ_e1vm2ePA3t9X6GBRk33kNO0rP_ketW7pJljsiKAkb1dkNyFgCKYL1LUH8xRFXYFCnsrbHV3LxWod5VbA/s640/20181030_232111.jpg" target="_blank">intl conf of surgeons - rk laxman</a><br>
	<a href="https://paulgraham.com/ambitious.html" target="_blank">frighteningly ambitious startup ideas</a><br>
	<a href="https://www.instagram.com/reels/DS9USsRE4ai/" target="_blank">adorable cat video</a><br>
	<a href="https://cdn.mos.cms.futurecdn.net/QfLVpW7bGiruhRX3TpLqna.jpeg" target="_blank">1970 nike manifesto</a><br>
	<a href="https://www.youtube.com/watch?v=eSvLFPFXjc8" target="_blank">field marshall sam manekshaw on leadership</a><br>
	<a href="https://www.instagram.com/reels/DJxgsSQy51D/" target="_blank">feel good inc mini cover</a><br>
	<br>
	PHOTOS I LIKE: <br>
	<div class="photo-masonry">
		<img src="/photos/puf89elf9uf81.jpg" alt="" />
		<img src="/photos/unnamed-2.jpg" alt="" />
		<img src="/photos/image7.jpeg" alt="" />
		<img src="/photos/First_flight2.jpg" alt="" />
		<img src="/photos/83549e13513a802a238631da9de651e0.jpg" alt="" />
		<img src="/photos/c90b19c991a299f57cdf6c0de0b3a900.jpg" alt="" />
		<img src="/photos/Nikola_Tesla.jpg" alt="" />
		<img src="/photos/Dolly_the_Sheep.jpg" alt="" />
		<img src="/photos/12856.jpg" alt="" />
	</div>
	<!--
	<br>
	LAST PLAYED SONG:

	{#if loading}
	  <p>Loading latest song...</p>
	{:else if error}
	  <p style="color: red;">Error: {error}</p>
	{:else}
	  <div class="song">
		<h1>{song.track.name} by {song.track.artist["#text"]}</h1>

	  </div>
	{/if}
	<br>
	-->


</div>

<style>
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
	.contact-link {
		color: #2563eb;
		text-decoration: underline;
		text-underline-offset: 2px;
		transition: color 0.2s;
	}
	.contact-link:hover {
		color: #1d4ed8;
		text-decoration-thickness: 2px;
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
	.lastfm-box {
		background: #f7f7f8;
		border-radius: 10px;
		border: 1px solid #e0e0e0;
		padding: 1rem 1.2rem;
		margin: 0 auto 2rem auto;
		max-width: 350px;
		text-align: center;
	}
	.album-art {
		width: 64px;
		height: 64px;
		object-fit: cover;
		border-radius: 6px;
		margin: 0.5rem 0;
	}
	.photo-masonry {
		column-count: 4;
		column-gap: 6px;
		margin-top: 0.5rem;
	}
	.photo-masonry img {
		width: 100%;
		height: auto;
		display: block;
		margin-bottom: 6px;
		break-inside: avoid;
	}
</style>
