<script lang="ts">
	import Nav from '$lib/Nav.svelte';
	import { onMount } from 'svelte';

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

	let qs: Q[] = $state([
		{ type: 'bool', q: 'Is this photo in color?', options: '', lo: 1, hi: 5, alo: '', ahi: '' },
		{ type: 'choice', q: 'What is the main subject?', options: 'airplane, car, boat, bird, kite', lo: 1, hi: 5, alo: '', ahi: '' },
		{ type: 'score', q: 'How crowded is the scene?', options: '', lo: 1, hi: 5, alo: 'empty', ahi: 'packed' },
		{ type: 'choice', q: 'What colour is the dog?', options: 'brown, black, white', lo: 1, hi: 5, alo: '', ahi: '' }
	]);
	let status = $state('connecting to the model…');
	let error = $state('');
	let timing = $state('');
	let imgMeta = $state('');
	let preview = $state('');
	let over = $state(false);
	let maxQ = $state(20);

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
			timing = `${n} question${n > 1 ? 's' : ''}, one forward pass · model ${Math.round(r.answer_ms)} ms · round trip ${Math.round(performance.now() - t0)} ms`;
		} catch (e) { error = (e as Error).message; }
		busy = false;
		if (again) { again = false; ask(); }
	}
	const askSoon = () => { clearTimeout(timer); timer = setTimeout(ask, 300); };

	async function loadImage(file: Blob | undefined | null) {
		if (!file || !file.type.startsWith('image/')) { error = "that file isn't an image."; return; }
		error = ''; imageKey = null; imgMeta = 'resizing…';
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
			imageB64 = await new Promise((res) => { const fr = new FileReader(); fr.onload = () => res(String(fr.result).split(',')[1]); fr.readAsDataURL(blob); });
			imgMeta = 'encoding…';
			const t0 = performance.now();
			const r = await upload();
			imgMeta = `${c.width}×${c.height} px · ${r.image_tokens} image tokens · upload + encode ${Math.round(performance.now() - t0)} ms`;
			qs.forEach((q) => (q.result = undefined));
			ask();
		} catch (e) { imgMeta = ''; error = (e as Error).message; }
	}

	function add() { if (qs.length < maxQ) qs.push({ type: 'bool', q: '', options: '', lo: 1, hi: 5, alo: '', ahi: '' }); }
	function remove(i: number) { qs.splice(i, 1); askSoon(); }
	function sorted(r: Result) { return r.type === 'score' ? r.answers : [...r.answers].sort((a, b) => b[1] - a[1]); }
	const pct = (p: number) => (p * 100).toFixed(1) + '%';

	onMount(() => {
		const paste = (e: ClipboardEvent) => { const f = e.clipboardData?.files[0]; if (f) loadImage(f); };
		document.addEventListener('paste', paste);
		const hash = new URLSearchParams(location.hash.slice(1));
		API = hash.get('api') || DEFAULT_API;
		token = hash.get('token') || '';
		(API ? api('/api/status') : Promise.reject())
			.then(async (s) => {
				maxQ = s.max_questions;
				status = `${s.model} + lora + typed heads · ${s.device}`;
				loadImage(await (await fetch(EXAMPLE)).blob());
			})
			.catch(() => {
				status = "offline right now. it runs on a single university gpu, so it isn't always up.";
				preview = EXAMPLE;
			});
		return () => document.removeEventListener('paste', paste);
	});
</script>

<svelte:head>
	<title>dragonfly · adit.run()</title>
	<meta name="description" content="ask an image many typed questions, get calibrated probabilities back in one forward pass." />
</svelte:head>

<div class="thin-centered">
	<Nav />
	<main>
		<h1>dragonfly</h1>
		<p>
			a vision-language model that answers every question about an image in <b>one forward pass</b>, without
			generating text. each answer is a calibrated probability distribution, and "can't answer" is its own
			probability. it's qwen3-vl-4b with lora and custom typed heads, trained on 740k questions.
		</p>
		<p class="muted">research preview. edit the questions, answers update as you type. nothing you upload is saved.</p>

		<p class="status">model: {status}</p>

		<label class="drop" class:over
			ondragenter={(e) => { e.preventDefault(); over = true; }}
			ondragover={(e) => { e.preventDefault(); over = true; }}
			ondragleave={() => (over = false)}
			ondrop={(e) => { e.preventDefault(); over = false; loadImage(e.dataTransfer?.files[0]); }}>
			{#if preview}<img src={preview} alt="your upload" />{:else}<span>drop an image, paste one, or click to choose</span>{/if}
			<input type="file" accept="image/*" hidden onchange={(e) => loadImage(e.currentTarget.files?.[0])} />
		</label>
		<p class="meta">{imgMeta}{#if imgMeta && preview}&nbsp;· {/if}{#if preview}click the image to change it{/if}</p>

		<p><b>QUESTIONS</b></p>
		{#each qs as q, i (q)}
			<div class="q" oninput={askSoon}>
				<div class="row">
					<select bind:value={q.type} onchange={() => { q.result = undefined; askSoon(); }} aria-label="answer type">
						<option value="bool">yes/no</option>
						<option value="choice">choice</option>
						<option value="score">scale</option>
					</select>
					<input class="grow" bind:value={q.q} maxlength="300" placeholder="ask something about the image" aria-label="question" />
					<button class="rm" onclick={() => remove(i)} aria-label="remove question">×</button>
				</div>
				{#if q.type === 'choice'}
					<input class="full" bind:value={q.options} placeholder="options, comma-separated (2–8)" aria-label="options" />
				{:else if q.type === 'score'}
					<div class="row scale">
						<input class="num" type="number" min="0" max="9" bind:value={q.lo} aria-label="scale low" />
						<input class="grow" bind:value={q.alo} maxlength="40" placeholder="low end means" aria-label="low label" />
						<input class="num" type="number" min="1" max="10" bind:value={q.hi} aria-label="scale high" />
						<input class="grow" bind:value={q.ahi} maxlength="40" placeholder="high end means" aria-label="high label" />
					</div>
				{/if}
				{#if q.result}
					{@const best = Math.max(...q.result.answers.map((a) => a[1]))}
					<div class="result">
						{#each sorted(q.result) as [name, p]}
							<div class="ans" class:top={p === best}>
								<span class="name">{name}</span>
								<span class="track"><span class="fill" style="width:{p * 100}%"></span></span>
								<span class="pct">{pct(p)}</span>
							</div>
						{/each}
						<span class="na" class:hi={q.result.na >= 0.5}>can't answer: {pct(q.result.na)}</span>
					</div>
				{/if}
			</div>
		{/each}
		<div class="row actions">
			<button onclick={add} disabled={qs.length >= maxQ}>+ add question</button>
			<span class="timing">{timing}</span>
		</div>
		{#if error}<p class="error">{error}</p>{/if}

		<p class="muted small">
			how: the image is encoded once; every question (and every option) is packed into the same sequence behind an
			attention mask that keeps them from seeing each other, so order never matters and each extra question costs
			~10 tokens. typed heads read the answers straight off the hidden states. limits: no reasoning step, so counting
			past ~5, small text and arithmetic are weak.
		</p>
	</main>
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
		box-sizing: border-box;
	}
	@media (max-width: 700px) {
		.thin-centered { width: auto; min-width: 0; margin: 1rem 16px 3rem; padding: 0; }
	}
	main { margin-top: 2.5rem; display: flex; flex-direction: column; gap: 0.9rem; }
	:global(html), :global(body) { font-family: 'JetBrains Mono', monospace; font-size: 14px; margin: 0; background: #fff; }
	h1 { font-size: 20px; font-weight: 700; margin: 0; }
	p { margin: 0; line-height: 1.6; }
	.muted { color: #666; }
	.small { font-size: 12px; margin-top: 1rem; }
	.status, .meta, .timing { font-size: 12px; color: #888; font-variant-numeric: tabular-nums; }
	.drop {
		display: flex; align-items: center; justify-content: center; min-height: 180px; padding: 0.75rem;
		border: 1px dashed #bbb; cursor: pointer; color: #888; text-align: center;
	}
	.drop.over { border-color: #2563eb; background: #eff6ff; }
	.drop img { max-width: 100%; max-height: 340px; display: block; }
	.q { border-top: 1px solid #eee; padding-top: 0.75rem; display: flex; flex-direction: column; gap: 0.5rem; }
	.row { display: flex; gap: 0.5rem; align-items: center; flex-wrap: wrap; }
	.grow { flex: 1 1 140px; min-width: 0; }
	.full { width: 100%; box-sizing: border-box; }
	.num { width: 3.5rem; }
	input, select, button {
		font: inherit; font-size: 13px; color: #222; background: #fff;
		border: 1px solid #ddd; padding: 0.35rem 0.5rem; border-radius: 0;
	}
	input:focus, select:focus { outline: none; border-color: #2563eb; }
	button { cursor: pointer; }
	button:hover:not(:disabled) { border-color: #222; }
	button:disabled { color: #aaa; cursor: default; }
	.rm { border: none; color: #888; padding: 0.35rem; }
	.result { display: flex; flex-direction: column; gap: 0.25rem; }
	.ans { display: grid; grid-template-columns: minmax(6rem, 11rem) 1fr 3.6rem; gap: 0.6rem; align-items: center; font-size: 13px; }
	.name { overflow-wrap: anywhere; }
	.track { height: 10px; background: #f3f4f6; }
	.fill { display: block; height: 100%; background: #bfdbfe; transition: width 0.3s ease; }
	.top .fill { background: #2563eb; }
	.top .name { font-weight: 700; }
	.pct { text-align: right; font-variant-numeric: tabular-nums; }
	.na { font-size: 12px; color: #888; }
	.na.hi { color: #b45309; font-weight: 700; }
	.actions { justify-content: space-between; }
	.error { color: #b91c1c; font-size: 13px; }
	@media (prefers-reduced-motion: reduce) { .fill { transition: none; } }
</style>
