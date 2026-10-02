<script lang="ts">
	import { onMount, tick } from 'svelte';

	// the model server: one P100 on CSC cPouta (Finland), HTTPS via Caddy. Empty until that VM exists; meanwhile
	// model/demo.sh opens this page with #api=http://localhost:8765&token=... (an SSH tunnel to a Roihu GPU).
	// The fragment never leaves the browser.
	const DEFAULT_API = '';
	let API = DEFAULT_API;
	let token = '';
	const EXAMPLE = '/photos/First_flight2.jpg';

	type Kind = 'bool' | 'choice' | 'score';
	type Result = { type: Kind; answers: [string, number][]; na: number };
	type Q = { type: Kind; q: string; options: string; lo: number; hi: number; alo: string; ahi: string; result?: Result };
	const blank = (): Q => ({ type: 'bool', q: '', options: '', lo: 1, hi: 5, alo: '', ahi: '' });
	const KINDS: [Kind, string][] = [['bool', 'yes / no'], ['choice', 'choice'], ['score', 'scale']];

	let qs: Q[] = $state([
		{ ...blank(), q: 'Is this photo in colour?' },
		{ ...blank(), type: 'choice', q: 'What is the main subject?', options: 'airplane, car, boat, bird, kite' },
		{ ...blank(), type: 'score', q: 'How crowded is the scene?', alo: 'empty', ahi: 'packed' },
		{ ...blank(), type: 'choice', q: 'What colour is the dog?', options: 'brown, black, white' }
	]);
	let online = $state(false);
	let status = $state('connecting…');
	let error = $state('');
	let timing = $state('');
	let preview = $state('');
	let over = $state(false);
	let maxQ = $state(20);
	let list: HTMLElement | undefined = $state();

	let imageKey: string | null = null;
	let imageB64 = '';
	let busy = false, again = false;
	let timer: ReturnType<typeof setTimeout>;

	async function api(path: string, body?: object) {
		// text/plain body: a "simple" CORS request, so no preflight round trip
		const url = API + path + (token ? `?token=${encodeURIComponent(token)}` : '');
		const r = await fetch(url, body ? { method: 'POST', body: JSON.stringify(body) } : {});
		const data = await r.json().catch(() => ({ error: `server error ${r.status}` }));
		if (!r.ok) throw new Error(data.error || `server error ${r.status}`);
		return data;
	}

	function complete(q: Q) {
		if (!q.q.trim()) return null;
		const out: Record<string, unknown> = { type: q.type, q: q.q.trim() };
		if (q.type === 'choice') {
			const opts = q.options.split(',').map((s) => s.trim()).filter(Boolean);
			if (new Set(opts.map((o) => o.toLowerCase())).size < 2) return null;
			out.options = opts;
		}
		if (q.type === 'score') {
			if (!(q.lo >= 0 && q.lo < q.hi && q.hi <= 10)) return null;
			out.scale = [q.lo, q.hi];
			out.anchors = [q.alo.trim(), q.ahi.trim()];
		}
		return out;
	}

	async function upload() {
		const r = await api('/api/image', { image: imageB64 });
		imageKey = r.key;
		return r;
	}

	async function ask() {
		if (!imageKey) return;
		if (busy) { again = true; return; } // latest edit wins
		const live = qs.map((q, i) => [i, complete(q)] as const).filter(([, c]) => c);
		if (!live.length) return;
		busy = true;
		error = '';
		const t0 = performance.now();
		try {
			const body = () => ({ key: imageKey, questions: live.map(([, c]) => c) });
			let r;
			try { r = await api('/api/answer', body()); }
			catch (e) { // the server keeps only recent images: re-send it once
				if (!String((e as Error).message).includes('upload it again')) throw e;
				await upload();
				r = await api('/api/answer', body());
			}
			r.results.forEach((res: Result, k: number) => { qs[live[k][0]].result = res; });
			const n = r.results.length;
			timing = `${n} question${n > 1 ? 's' : ''} in one pass · ${Math.round(r.answer_ms)} ms`;
		} catch (e) { error = (e as Error).message; }
		busy = false;
		if (again) { again = false; ask(); }
	}
	const askSoon = () => { clearTimeout(timer); timer = setTimeout(ask, 300); };

	async function loadImage(file: Blob | undefined | null) {
		if (!file || !file.type.startsWith('image/')) { error = "That file isn't an image."; return; }
		error = ''; imageKey = null;
		try {
			const bmp = await createImageBitmap(file, { imageOrientation: 'from-image' });
			// the model sees at most 448×448 pixels' worth (aspect kept): send exactly that
			const s = Math.min(1, Math.sqrt((448 * 448) / (bmp.width * bmp.height)));
			const c = document.createElement('canvas');
			c.width = Math.round(bmp.width * s); c.height = Math.round(bmp.height * s);
			c.getContext('2d')!.drawImage(bmp, 0, 0, c.width, c.height);
			const blob: Blob = await new Promise((res) => c.toBlob((b) => res(b!), 'image/jpeg', 0.9));
			if (preview.startsWith('blob:')) URL.revokeObjectURL(preview);
			preview = URL.createObjectURL(blob);
			if (!online) return;
			imageB64 = await new Promise((res) => { const fr = new FileReader(); fr.onload = () => res(String(fr.result).split(',')[1]); fr.readAsDataURL(blob); });
			await upload();
			qs.forEach((q) => (q.result = undefined));
			ask();
		} catch (e) { error = (e as Error).message; }
	}

	async function add() {
		if (qs.length >= maxQ) return;
		qs.push(blank());
		await tick();
		list?.querySelector<HTMLInputElement>('.q:last-child .text')?.focus();
	}
	function remove(i: number) { qs.splice(i, 1); askSoon(); }
	function sorted(r: Result) { return r.type === 'score' ? r.answers : [...r.answers].sort((a, b) => b[1] - a[1]); }
	const pct = (p: number) => Math.round(p * 100) + '%';

	onMount(() => {
		const paste = (e: ClipboardEvent) => { const f = e.clipboardData?.files[0]; if (f) loadImage(f); };
		document.addEventListener('paste', paste);
		const hash = new URLSearchParams(location.hash.slice(1));
		API = hash.get('api') || DEFAULT_API;
		token = hash.get('token') || '';
		(API ? api('/api/status') : Promise.reject())
			.then(async (s) => {
				maxQ = s.max_questions;
				online = true;
				status = '';
				loadImage(await (await fetch(EXAMPLE)).blob());
			})
			.catch(() => {
				status = "The model is offline right now. It runs on a single university GPU, so it isn't always up.";
				preview = EXAMPLE;
			});
		return () => document.removeEventListener('paste', paste);
	});
</script>

<svelte:head>
	<title>Dragonfly (Research Preview)</title>
	<meta name="description" content="Ask an image many typed questions and get calibrated probabilities back from one forward pass." />
</svelte:head>

<div class="page">
	<header>
		<h1>Dragonfly <span>(Research Preview)</span></h1>
	</header>

	<div class="split">
		<section class="left">
			<label class="square" class:over class:empty={!preview}
				ondragenter={(e) => { e.preventDefault(); over = true; }}
				ondragover={(e) => { e.preventDefault(); over = true; }}
				ondragleave={() => (over = false)}
				ondrop={(e) => { e.preventDefault(); over = false; loadImage(e.dataTransfer?.files[0]); }}>
				{#if preview}<img src={preview} alt="What the questions are about" />{:else}<span class="hint">Drop an image</span>{/if}
				<input type="file" accept="image/*" hidden onchange={(e) => loadImage(e.currentTarget.files?.[0])} />
			</label>
			<p class="caption">
				{#if status}{status}{:else}Drop, paste or click to change the image.{/if}
			</p>
		</section>

		<section class="right" bind:this={list}>
			{#each qs as q, i (q)}
				<div class="q" oninput={askSoon}>
					<div class="line">
						<input class="text" bind:value={q.q} maxlength="300" placeholder="Ask something about the image" aria-label="Question" />
						<button class="rm" onclick={() => remove(i)} aria-label="Remove question">×</button>
					</div>
					<div class="kinds" role="radiogroup" aria-label="Answer type">
						{#each KINDS as [k, label]}
							<button role="radio" aria-checked={q.type === k} class:on={q.type === k}
								onclick={() => { q.type = k; q.result = undefined; askSoon(); }}>{label}</button>
						{/each}
					</div>
					{#if q.type === 'choice'}
						<input class="sub" bind:value={q.options} placeholder="Options, separated by commas" aria-label="Options" />
					{:else if q.type === 'score'}
						<div class="scale">
							<input class="num" type="number" min="0" max="9" bind:value={q.lo} aria-label="Scale low" />
							<input class="sub" bind:value={q.alo} maxlength="40" placeholder="means" aria-label="Low label" />
							<span>to</span>
							<input class="num" type="number" min="1" max="10" bind:value={q.hi} aria-label="Scale high" />
							<input class="sub" bind:value={q.ahi} maxlength="40" placeholder="means" aria-label="High label" />
						</div>
					{/if}
					{#if q.result}
						{@const best = Math.max(...q.result.answers.map((a) => a[1]))}
						<div class="bars">
							{#each sorted(q.result) as [name, p]}
								<div class="bar" class:top={p === best}>
									<span class="name">{name}</span>
									<span class="track"><span class="fill" style="width:{p * 100}%"></span></span>
									<span class="pct">{pct(p)}</span>
								</div>
							{/each}
							<p class="na" class:hi={q.result.na >= 0.5}>can't answer · {pct(q.result.na)}</p>
						</div>
					{/if}
				</div>
			{/each}
			<button class="plus" onclick={add} disabled={qs.length >= maxQ} aria-label="Add a question">+</button>
			{#if timing || error}<p class="caption" class:err={error}>{error || timing}</p>{/if}
		</section>
	</div>

	<p class="foot"><a href="/">adit.run</a></p>
</div>

<style>
	/* a plain hand-made HTML page: the browser's own Times and Courier, link blue, black on white */
	.page {
		--ink: #000;
		--muted: #707070;
		--line: #d0d0d0;
		--link: #0000ee;
		--na: #a0522d;
		--mono: 'Courier New', Courier, monospace;
		min-height: 100vh;
		box-sizing: border-box;
		padding: 2rem 4vw 3rem;
		background: #fff;
		color: var(--ink);
		font-family: 'Times New Roman', Times, serif;
		font-size: 18px;
		line-height: 1.4;
	}
	h1 { margin: 0 0 2.5rem; font-size: 1.6rem; font-weight: bold; }
	h1 span { font-weight: normal; color: var(--muted); }
	.foot { margin: 4rem 0 0; font-size: 0.85rem; }
	.foot a { color: var(--link); }
	.split { display: grid; grid-template-columns: 1fr 1fr; gap: 4vw; align-items: start; }
	.left { position: sticky; top: 2rem; }
	@media (max-width: 760px) {
		.split { grid-template-columns: 1fr; gap: 2.5rem; }
		.page { padding-inline: 16px; }
		.left { position: static; }
	}
	.square {
		display: flex; align-items: center; justify-content: center;
		width: 100%; max-width: 80vh; aspect-ratio: 1 / 1; box-sizing: border-box;
		border: 1px solid var(--line); cursor: pointer; transition: border-color 0.2s;
	}
	.square.empty { border-style: dashed; }
	.square:hover, .square.over { border-color: var(--ink); }
	.square img { width: 100%; height: 100%; object-fit: contain; display: block; }
	.square .hint { color: var(--muted); }
	.caption { margin: 0.75rem 0 0; color: var(--muted); font-size: 0.85rem; max-width: 80vh; }
	.caption.err { color: #a12a1e; }

	.right { display: flex; flex-direction: column; }
	.q { padding: 0 0 1.6rem; margin-bottom: 1.6rem; border-bottom: 1px solid var(--line); display: flex; flex-direction: column; gap: 0.45rem; }
	.line { display: flex; align-items: baseline; gap: 0.5rem; }
	input {
		font: inherit; color: var(--ink); background: transparent; border: 0; border-radius: 0;
		padding: 0.1rem 0; min-width: 0; outline: none;
	}
	input::placeholder { color: #b5b5b5; }
	.text { flex: 1; font-size: 1.15rem; }
	.sub { font-size: 0.95rem; border-bottom: 1px solid var(--line); }
	.sub:focus, .num:focus { border-bottom-color: var(--ink); }
	.scale { display: flex; align-items: baseline; gap: 0.5rem; font-size: 0.95rem; color: var(--muted); }
	.scale .sub { flex: 1; }
	.num { width: 2.2rem; text-align: center; border-bottom: 1px solid var(--line); font-size: 0.95rem; appearance: textfield; -moz-appearance: textfield; }
	.num::-webkit-inner-spin-button, .num::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
	button { font: inherit; color: inherit; background: none; border: 0; padding: 0; cursor: pointer; }
	.rm { color: #c4c4c4; font-size: 1.2rem; line-height: 1; }
	.rm:hover { color: var(--ink); }
	/* answer types read like links; the chosen one is plain black text */
	.kinds { display: flex; gap: 1rem; font-size: 0.9rem; }
	.kinds button { color: var(--link); text-decoration: underline; }
	.kinds button.on { color: var(--ink); text-decoration: none; }

	.bars { display: flex; flex-direction: column; gap: 0.2rem; margin-top: 0.4rem; font-size: 0.95rem; }
	.bar { display: grid; grid-template-columns: minmax(5rem, 9rem) 1fr 2.8rem; gap: 0.9rem; align-items: center; color: var(--muted); }
	.bar.top { color: var(--ink); }
	.name { overflow-wrap: anywhere; }
	.track { height: 1px; background: var(--line); position: relative; }
	.fill { position: absolute; left: 0; top: -1px; height: 3px; background: #c9c9c9; transition: width 0.35s ease; }
	.top .fill { background: var(--ink); }
	.pct { text-align: right; font-family: var(--mono); font-size: 0.9rem; }
	.na { margin: 0.2rem 0 0; color: #555; }
	.na, .caption { font-family: var(--mono); font-size: 0.8rem; } /* Courier sets light: a touch larger and darker */
	.caption:not(.err) { color: #555; }
	.na.hi { color: var(--na); }

	.plus {
		width: 100%; height: 5.5rem; border: 1px solid var(--line); color: var(--muted);
		font-size: 2.4rem; line-height: 1; transition: border-color 0.2s, color 0.2s;
	}
	.plus:hover:not(:disabled) { border-color: var(--ink); color: var(--ink); }
	.plus:disabled { opacity: 0.4; cursor: default; }
	@media (prefers-reduced-motion: reduce) { .fill, .square, .plus { transition: none; } }
</style>
