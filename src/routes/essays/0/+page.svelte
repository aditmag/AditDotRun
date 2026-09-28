<script>
	import Nav from '$lib/Nav.svelte';
</script>

<div class="thin-centered">
	<Nav />

    <br/><br/>
    <center>
        <h1><span style="color: #e06666">f'(x) = 0</span>: OPTIMIZING FOR 'BARE MINIMUM'</h1>

        <br><br>
        
        <i>
        “ticking away the moments that make up a dull day <br>
        fritter and waste the hours in an offhand way <br>
        kicking around on a piece of ground in your hometown <br>
        waiting for someone or something to show you the way” <br>
        – pink floyd
        </i>
        <br><br>
    </center>

    <span style="color: #e06666">f’(x) = 0</span> is the point where nothing moves. there’s no slope, no force, no climbing, and no falling. you’re parked at a local optimum. <br><br>
the funny part is that getting here is nothing more than an optimization problem over your life. <br><br>

most of us have grown up listening to the ‘whatever you do, put your heart and soul into it’. this advice makes a pretty dangerous assumption: the output scales linearly with input i.e everything you do will be productive. to optimize for a goal, let’s break this down. consider each major part of your life to be a function of time and energy: <br><br>

<span style="color: #e06666">job(t, e) := </span> ... // work <br>
<span style="color: #e06666">edu(t, e) := </span> ... // school <br>
<span style="color: #e06666">fam(t, e) := </span> ... // family, friends, outings <br>
<span style="color: #e06666">exe(t, e) := </span> ... // gym, sport, running <br>
<span style="color: #e06666">hob(t, e) := </span> ... // literally whatever <br><br>

the task is to optimize these functions based on your goals. <br><br>

example: are you grinding courses at uni when all you dream about is building the next big thing? <br><br>

<span style="color: #e06666">max(hob)</span> and optimize <span style="color: #e06666">edu</span> to a goal that makes sense. <br><br>

<i>
“for example, i always used to say that the smartest student will get 85% in all of his courses. this way, you end up with somewhere around 4.0 score, but you did not over-study, and you did not under-study.”
- andrej karpathy <br><br>
</i>

it’s completely fine to optimize some functions to achieve a bare minimum in order to maximize the central goal - finishing more credits at uni early on to take a semester off and travel, or doing the absolute bare minimum at work because your goal is to spend as much time as you can with your family in the evening. <br><br>

<span style="color: #e06666">f’(x) = 0</span> isn’t being a loser and giving up. it’s simply a way to optimize for the end goal. <br><br>

––– <br><br>

i know this is some fairly obvious stuff wrapped in [perhaps unnecessarily] technical jargon but breaking down life into functions that can be optimized really changed the perspective for me. <br><br>

the beauty of this point of view is that it can be broken down even further and allows you to explicitly define thresholds and help you decide how much time and energy to give each function.



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
</style>
