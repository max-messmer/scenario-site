<script lang="ts">
	import { onMount } from 'svelte';

	let cursorX = $state(0);
	let cursorY = $state(0);
	let cursorHovering = $state(false);
	let cursorVisible = $state(false);
	let scrollY = $state(0);
	let innerHeight = $state(0);
	let innerWidth = $state(0);
	let loaded = $state(false);
	let mobileMenuOpen = $state(false);

	function toggleMobileMenu() {
		mobileMenuOpen = !mobileMenuOpen;
	}

	function mobileNavTo(id: string) {
		mobileMenuOpen = false;
		scrollToSection(id);
	}

	// Lyric parallax
	let lyricSection: HTMLElement | undefined = $state(undefined);
	let lyricOffset = $state(0); // how far section center is from viewport center (-1 to 1 ish)

	// Three rows of lyrics — some words drift on x-axis when scrolling
	const lyricRows = [
		[
			{ text: 'OUR',   xSpeed: 0 },
			{ text: 'LOVED', xSpeed: -0.12, cls: 'lw-accent' },
			{ text: 'ONES',  xSpeed: 0 },
			{ text: 'MOURN', xSpeed: 0.08, cls: 'lw-muted' },
		],
		[
			{ text: 'ON',      xSpeed: 0.1 },
			{ text: 'WINTER',  xSpeed: 0, cls: 'lw-accent' },
			{ text: 'SHORES',  xSpeed: -0.06 },
			{ text: 'FOR',     xSpeed: 0, cls: 'lw-dim' },
			{ text: 'DEATH',   xSpeed: 0 },
		],
		[
			{ text: 'IS',      xSpeed: 0, cls: 'lw-dim' },
			{ text: 'CERTAIN', xSpeed: 0.1 },
			{ text: 'ONCE',    xSpeed: 0 },
			{ text: "YOU'RE",  xSpeed: -0.08, cls: 'lw-muted' },
			{ text: 'BORN',    xSpeed: 0, cls: 'lw-accent' },
		],
	];

	function handleMouseMove(e: MouseEvent) {
		cursorX = e.clientX;
		cursorY = e.clientY;
		cursorVisible = true;
	}

	let cursorHidden = $state(false);

	function handleMouseEnterLink() {
		cursorHovering = true;
	}

	function handleMouseLeaveLink() {
		cursorHovering = false;
	}

	function handleEmbedEnter() {
		cursorHidden = true;
	}

	function handleEmbedLeave() {
		cursorHidden = false;
	}

	function scrollToSection(id: string) {
		const el = document.getElementById(id);
		if (el) el.scrollIntoView({ behavior: 'smooth' });
	}

	// Compute how far the lyric section center is from viewport center (in px)
	$effect(() => {
		const _s = scrollY;
		const _h = innerHeight;
		if (lyricSection && _h) {
			const rect = lyricSection.getBoundingClientRect();
			const sectionCenter = rect.top + rect.height / 2;
			const vpCenter = _h / 2;
			lyricOffset = sectionCenter - vpCenter; // positive = section below center, negative = above
		}
	});

	function wordXTransform(xSpeed: number): string {
		if (xSpeed === 0 || innerWidth < 601) return '';
		const x = lyricOffset * xSpeed;
		return `transform: translateX(${x}px)`;
	}

	onMount(() => {
		loaded = true;

		const observer = new IntersectionObserver(
			(entries) => {
				entries.forEach((entry) => {
					if (entry.isIntersecting) {
						entry.target.classList.add('in-view');
					}
				});
			},
			{ threshold: 0.08, rootMargin: '0px 0px -30px 0px' }
		);

		document.querySelectorAll('[data-reveal]').forEach((el) => observer.observe(el));

		const interactiveEls = document.querySelectorAll('a, button, [data-hover]');
		interactiveEls.forEach((el) => {
			el.addEventListener('mouseenter', handleMouseEnterLink);
			el.addEventListener('mouseleave', handleMouseLeaveLink);
		});

		const embedEls = document.querySelectorAll('.release-embed iframe, .watch-embed iframe');
		embedEls.forEach((el) => {
			el.addEventListener('mouseenter', handleEmbedEnter);
			el.addEventListener('mouseleave', handleEmbedLeave);
		});

		return () => {
			observer.disconnect();
			interactiveEls.forEach((el) => {
				el.removeEventListener('mouseenter', handleMouseEnterLink);
				el.removeEventListener('mouseleave', handleMouseLeaveLink);
			});
			embedEls.forEach((el) => {
				el.removeEventListener('mouseenter', handleEmbedEnter);
				el.removeEventListener('mouseleave', handleEmbedLeave);
			});
		};
	});
</script>

<svelte:window bind:scrollY bind:innerHeight bind:innerWidth onmousemove={handleMouseMove} />

<!-- CURSOR -->
<div
	class="cursor"
	class:cursor-visible={cursorVisible}
	class:cursor-hovering={cursorHovering}
	class:cursor-hidden={cursorHidden}
	style="transform: translate({cursorX}px, {cursorY}px)"
></div>
<div
	class="cursor-dot"
	class:cursor-visible={cursorVisible}
	class:cursor-hovering={cursorHovering}
	class:cursor-hidden={cursorHidden}
	style="transform: translate({cursorX}px, {cursorY}px)"
></div>

<!-- GRAIN -->
<div class="grain"></div>

<!-- TOP BAR -->
<header class="topbar" class:loaded>
	<div class="topbar-left">
		<button class="topbar-logo" class:menu-open={mobileMenuOpen} onclick={() => mobileNavTo('top')} data-hover>SCENARIO<span class="dot">.</span></button>
	</div>
	<nav class="topbar-nav">
		<button onclick={() => scrollToSection('music')} data-hover>RELEASES</button>
		<button onclick={() => scrollToSection('watch')} data-hover>WATCH</button>
		<button onclick={() => scrollToSection('contact')} data-hover>FIND US</button>
	</nav>
	<!-- Mobile hamburger -->
	<button class="mobile-menu-btn" class:active={mobileMenuOpen} onclick={toggleMobileMenu} aria-label="Menu">
		<span class="bar"></span>
		<span class="bar"></span>
		<span class="bar"></span>
	</button>
</header>

<!-- MOBILE MENU OVERLAY -->
<div class="mobile-menu" class:open={mobileMenuOpen}>
	<nav class="mobile-menu-nav">
		<button onclick={() => mobileNavTo('music')}>RELEASES</button>
		<button onclick={() => mobileNavTo('watch')}>WATCH</button>
		<button onclick={() => mobileNavTo('contact')}>FIND US</button>
	</nav>
</div>

<!-- ====== CONTENT START ====== -->
<main id="top" class:loaded>

	<!-- OPENING BLOCK -->
	<section class="opener">
		<div class="opener-title" data-reveal>
			<h1>SCEN<br class="desktop-only"/>ARIO<span class="dot">.</span></h1>
		</div>
		<div class="opener-image opener-image-1" data-reveal>
			<img src="/photos/4.jpeg" alt="Live show energy" />
		</div>
	</section>

	<!-- SPLIT BLOCK — woods photo meets opener bottom edge -->
	<section class="split-block" data-reveal>
		<div class="split-image">
			<img src="/photos/7.webp" alt="Band in the woods" />
		</div>
		<div class="split-text">
			<p class="split-headline mb-2">STARS ARE<br class="desktop-only"/> GOING BLACK<br class="desktop-only"/><br class="desktop-only"/> AND THEY<br class="desktop-only"/> AREN'T COMING BACK.</p>
      <span class="eyebrow">About</span>
			<div class="rule"></div>
			<p class="split-body">
				Scenario. is a band from Cincinnati, Ohio. They are a 4 piece band that plays a mix of post-hardcore, screamo, and punk. They are known for their raw, unfiltered, and unapologetic sound. They are currently working on their new album, which is set to be released in 2026. They are currently touring the US and Europe. All of this information was written by AI so don't take it too seriously. Get in touch if you'd like to book us for a show. Listen to our music on <a class="split-body-link" href="https://open.spotify.com/artist/15hzwr8BhuDVJfhpOGKy3e?si=NIh5fYcaRZe69VOKlOxrDg" target="_blank" rel="noopener" data-hover>Spotify</a>. 
			</p>
		</div>
    <div class="opener-image opener-image-2" data-reveal>
			<img src="/photos/1.jpeg" alt="Guitarist performing" />
		</div>
	</section>

	<!-- LYRIC SECTION — three rows, x-drift on scroll -->
	<section class="lyric-section" bind:this={lyricSection}>
		{#each lyricRows as row, i}
			<div class="lyric-row lyric-row-{i}">
				{#each row as w}
					<span class="lyric-word {w.cls || ''}" style={wordXTransform(w.xSpeed)}>{w.text}</span>
				{/each}
			</div>
		{/each}
	</section>

	<!-- RELEASES -->
	<section id="music" class="releases" data-reveal>
		<div class="releases-header">
			<span class="eyebrow">RELEASES</span>
			<h2 class="releases-title">LISTEN</h2>
		</div>
		<div class="releases-grid">
			<div class="release-embed">
				<iframe
					title="When All is Said and Done by SCENARIO."
					src="https://open.spotify.com/embed/album/4SlW9W26QvRsVPrKFTkyGP?utm_source=generator"
					allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
					loading="lazy"
				></iframe>
				<a href="https://open.spotify.com/album/4SlW9W26QvRsVPrKFTkyGP" target="_blank" rel="noopener" class="release-link" data-hover>WHEN ALL IS SAID AND DONE — SPOTIFY ↗</a>
			</div>
			<div class="release-embed">
				<iframe
					title="Sounds in Sequence by SCENARIO."
					src="https://open.spotify.com/embed/album/0gpvNpkZHeLAbiXg1ibP3h?utm_source=generator"
					allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
					loading="lazy"
				></iframe>
				<a href="https://open.spotify.com/album/0gpvNpkZHeLAbiXg1ibP3h" target="_blank" rel="noopener" class="release-link" data-hover>SOUNDS IN SEQUENCE — SPOTIFY ↗</a>
			</div>
		</div>
	</section>

	<!-- WATCH -->
	<section id="watch" class="watch-section" data-reveal>
		<div class="watch-header">
			<span class="eyebrow">VIDEOS</span>
			<h2 class="watch-title">WATCH</h2>
		</div>
		<div class="watch-grid">
			<div class="watch-embed">
				<iframe
					src="https://www.youtube.com/embed/KHgq6fJDni0?si=XBrr-v4FpRMgY2XV"
					title="YouTube video player"
					frameborder="0"
					allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
					referrerpolicy="strict-origin-when-cross-origin"
					allowfullscreen
				></iframe>
			</div>
			<div class="watch-embed">
				<iframe
					src="https://www.youtube.com/embed/GLBm_Y9Dbpo?si=gkJRAn8wkyq9OQiv"
					title="YouTube video player"
					frameborder="0"
					allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
					referrerpolicy="strict-origin-when-cross-origin"
					allowfullscreen
				></iframe>
			</div>
		</div>
	</section>

	<!-- FIND US -->
	<section id="contact" class="findus-section" data-reveal>
		<div class="findus-header">
			<span class="eyebrow">LINKS</span>
			<h2 class="findus-title">FIND US</h2>
		</div>
		<div class="findus-links">
			<a href="https://www.instagram.com/scenariopunk" target="_blank" rel="noopener" data-hover>INSTAGRAM ↗</a>
			<a href="https://open.spotify.com/artist/15hzwr8BhuDVJfhpOGKy3e?si=NIh5fYcaRZe69VOKlOxrDg" target="_blank" rel="noopener" data-hover>SPOTIFY ↗</a>
			<a href="https://music.apple.com/us/artist/scenario/1623523866" target="_blank" rel="noopener" data-hover>APPLE MUSIC ↗</a>
			<a href="https://scenariopunk.bandcamp.com" target="_blank" rel="noopener" data-hover>BANDCAMP ↗</a>
			<a href="#" data-hover>BOOKING ↗</a>
		</div>
	</section>

	<!-- FULL WIDTH PHOTO -->
	<section class="fullwidth-photo" data-reveal>
		<img src="/photos/8.jpg" alt="Scenario band" />
	</section>

</main>

<style>
  .mb-2 {
    margin-bottom: 2rem;
  }
	/* ============================================
	   CURSOR (hidden on mobile, shown 601px+)
	   ============================================ */
	.cursor {
		display: none;
		position: fixed;
		top: -20px;
		left: -20px;
		width: 40px;
		height: 40px;
		border: 1px solid #cfccd6;
		border-radius: 50%;
		pointer-events: none;
		z-index: 10000;
		mix-blend-mode: difference;
		transition: width 0.35s cubic-bezier(0.16, 1, 0.3, 1),
					height 0.35s cubic-bezier(0.16, 1, 0.3, 1),
					top 0.35s cubic-bezier(0.16, 1, 0.3, 1),
					left 0.35s cubic-bezier(0.16, 1, 0.3, 1),
					background 0.35s ease,
					border 0.35s ease,
					opacity 0.3s ease;
		opacity: 0;
		will-change: transform;
		background: transparent;
	}
	@media (min-width: 601px) {
		.cursor { display: block; }
	}

	.cursor.cursor-hovering {
		width: 50px;
		height: 50px;
		top: -25px;
		left: -25px;
		background: #cfccd6;
		border-color: transparent;
	}

	.cursor-dot {
		display: none;
		position: fixed;
		top: -3px;
		left: -3px;
		width: 6px;
		height: 6px;
		background: #cfccd6;
		border-radius: 50%;
		pointer-events: none;
		z-index: 10001;
		mix-blend-mode: difference;
		transition: opacity 0.25s ease, width 0.35s ease, height 0.35s ease, top 0.35s ease, left 0.35s ease;
		opacity: 0;
		will-change: transform;
	}
	@media (min-width: 601px) {
		.cursor-dot { display: block; }
	}

	.cursor-dot.cursor-hovering {
		width: 50px;
		height: 50px;
		top: -25px;
		left: -25px;
		background: rgba(164, 66, 0, 0.3);
		border-radius: 50%;
	}

	.cursor-visible { opacity: 1; }
	.cursor-hidden { opacity: 0 !important; }

	/* ============================================
	   GRAIN
	   ============================================ */
	.grain {
		position: fixed;
		inset: -150%;
		width: 400%;
		height: 400%;
		z-index: 9999;
		pointer-events: none;
		opacity: 0.08;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='1.2' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
		animation: grain 0.15s steps(2) infinite;
	}
	@keyframes fadeIn {
		from { opacity: 0; }
		to { opacity: 1; }
	}

	@keyframes grain {
		0%   { transform: translate(0, 0); }
		10%  { transform: translate(-8%, -3%); }
		20%  { transform: translate(6%, 7%); }
		30%  { transform: translate(-12%, 2%); }
		40%  { transform: translate(3%, -11%); }
		50%  { transform: translate(-5%, 9%); }
		60%  { transform: translate(10%, -6%); }
		70%  { transform: translate(-3%, 13%); }
		80%  { transform: translate(8%, -4%); }
		90%  { transform: translate(-7%, 5%); }
		100% { transform: translate(0, 0); }
	}

	/* ============================================
	   TOPBAR
	   ============================================ */
	.topbar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 1000;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1rem 2rem;
		opacity: 0;
		transform: translateY(-10px);
		transition: all 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.2s;
	}
  @media (min-width: 901px) {
    .topbar {
      background: rgba(0, 23, 31, 0.4);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
    }
  }

	.topbar.loaded {
		opacity: 1;
		transform: translateY(0);
	}

	.topbar-logo {
		display: none;
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: 1.1rem;
		color: #cfccd6;
		background: none;
		border: none;
		cursor: pointer;
		letter-spacing: 0.15em;
	}
	.topbar-logo.menu-open {
		display: block;
		animation: fadeIn 0.3s ease forwards;
	}
	@media (min-width: 601px) {
		.topbar-logo { display: block; cursor: none; }
	}

	.topbar-nav {
		display: none;
	}
	@media (min-width: 601px) {
		.topbar-nav {
			display: flex;
			gap: 2rem;
		}
	}

	.topbar-nav button {
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 1rem;
		font-weight: 500;
		color: #fff;
		background: none;
		border: none;
		cursor: none;
		letter-spacing: 0.25em;
		text-transform: uppercase;
		transition: opacity 0.3s ease;
	}

	/* ---- Mobile hamburger ---- */
	.mobile-menu-btn {
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: 5px;
		background: rgba(0, 23, 31, 0.4);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		border: none;
		cursor: pointer;
		padding: .6rem;
		z-index: 1002;
	}
	@media (min-width: 601px) {
		.mobile-menu-btn { display: none; }
	}

	.mobile-menu-btn .bar {
		display: block;
		width: 28px;
		height: 2px;
		background: #cfccd6;
		transition: transform 0.3s ease, opacity 0.3s ease;
		transform-origin: center;
	}

	.mobile-menu-btn.active {
		background: transparent;
		backdrop-filter: none;
		-webkit-backdrop-filter: none;
	}

	.mobile-menu-btn.active .bar:nth-child(1) {
		transform: translateY(6.5px) rotate(45deg);
	}
	.mobile-menu-btn.active .bar:nth-child(2) {
		opacity: 0;
	}
	.mobile-menu-btn.active .bar:nth-child(3) {
		transform: translateY(-6.5px) rotate(-45deg);
	}

	/* ---- Mobile menu overlay ---- */
	.mobile-menu {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 999;
		background: rgba(0, 23, 31, 0.5);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		padding: 4rem 2rem 2rem;
		opacity: 0;
		pointer-events: none;
		transform: translateY(-10px);
		transition: opacity 0.3s ease, transform 0.3s ease;
	}
	@media (min-width: 601px) {
		.mobile-menu { display: none; }
	}

	.mobile-menu.open {
		opacity: 1;
		pointer-events: auto;
		transform: translateY(0);
	}

	.mobile-menu-nav {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		gap: 0;
	}

	.mobile-menu-nav button {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: 1.8rem;
		color: #cfccd6;
		background: none;
		border: none;
		letter-spacing: 0.05em;
		line-height: 1.4;
		padding: 0.75rem 0;
		width: 100%;
		text-align: left;
		cursor: pointer;
		transition: color 0.3s ease;
	}

	.mobile-menu-nav button:hover {
		color: #a44200;
	}

	/* ============================================
	   MAIN LOAD
	   ============================================ */
	main {
		opacity: 0;
		transition: opacity 0.6s ease 0.1s;
	}
	main.loaded { opacity: 1; }

	/* ============================================
	   REVEAL SYSTEM
	   ============================================ */
	:global([data-reveal]) {
		opacity: 0;
		transform: translateY(40px);
		transition: opacity 0.9s cubic-bezier(0.16, 1, 0.3, 1),
					transform 0.9s cubic-bezier(0.16, 1, 0.3, 1);
	}
	:global([data-reveal].in-view) {
		opacity: 1;
		transform: translateY(0);
	}

	/* ============================================
	   EYEBROW + RULE + HELPERS
	   ============================================ */
	.eyebrow {
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 1rem;
		font-weight: 500;
		letter-spacing: 0.4em;
		color: #a44200;
		text-transform: uppercase;
		display: block;
		margin: 0.6rem;
	}

	.rule {
		width: 100%;
		height: 1px;
		background: #69140e;
		margin: 0 0 1rem;
	}

	.dot { color: #a44200; }

	.desktop-only {
		display: none;
	}
	@media (min-width: 901px) {
		.desktop-only { display: block; }
	}

	/* ============================================
	   OPENER
	   ============================================ */
	.opener {
		position: relative;
		display: grid;
		grid-template-columns: 1fr;
		grid-template-rows: auto auto;
		padding: 0;
		gap: 0;
		overflow: hidden;
	}
	@media (min-width: 901px) {
		.opener {
			grid-template-columns: 1fr 1fr;
			padding: 4rem 0 0 2rem;
		}
	}

	.opener-title {
		grid-column: 1;
		grid-row: 1;
		z-index: 3;
		display: flex;
		align-items: flex-start;
		padding: 4rem 0 0 2rem;
	}
	@media (min-width: 901px) {
		.opener-title {
			grid-column: 1 / 2;
			grid-row: 1 / 3;
      padding: 6rem 0 0 2rem;
		}
	}

	.opener-title h1 {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: clamp(5rem, 18vw, 10rem);
		line-height: 0.82;
		letter-spacing: -0.02em;
		color: #cfccd6;
		position: relative;
		z-index: 3;
	}
	@media (min-width: 901px) {
		.opener-title h1 {
      font-size: clamp(6rem, 23vw, 30rem);
      letter-spacing: -.8vw;
      transform: scaleX(1.25) translateX(3vw);
		}
	}

	.opener-image {
		overflow: hidden;
		position: relative;
	}

	.opener-image img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		filter: grayscale(20%) contrast(1.15) brightness(0.85);
		transition: transform 1.2s cubic-bezier(0.16, 1, 0.3, 1), filter 0.6s ease;
	}

	.opener-image img:hover {
		transform: scale(1.04);
		filter: grayscale(0%) contrast(1.1) brightness(0.9);
	}

	.opener-image-1 {
		grid-column: 1;
		grid-row: 2;
		height: 300px;
		margin-left: 0;
		margin-top: -3rem;
		z-index: 2;
	}
	@media (min-width: 901px) {
		.opener-image-1 {
			grid-column: 2 / 3;
			grid-row: 1 / 2;
			height: 65vh;
			margin-left: -8vw;
			margin-top: 0;
		}
	}

	.opener-image-2 {
		grid-column: 1;
		grid-row: 3;
		width: 60%;
		height: 30vh;
		margin-top: -2rem;
		z-index: 1;
		justify-self: end;
	}
	@media (min-width: 901px) {
		.opener-image-2 {
			grid-column: auto;
			grid-row: auto;
			width: auto;
			height: 35vh;
			margin-top: 0;
		}
	}

	/* ============================================
	   SPLIT BLOCK
	   ============================================ */
	.split-block {
		display: grid;
		grid-template-columns: 1fr;
		min-height: auto;
		position: relative;
		margin-top: 0;
		z-index: 4;
	}
	@media (min-width: 901px) {
		.split-block {
			grid-template-columns: 2fr 1fr 1fr;
		}
	}

	.split-image {
		position: relative;
		overflow: hidden;
		height: 300px;
	}
	@media (min-width: 901px) {
		.split-image { height: auto; }
	}

	.split-image img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		filter: grayscale(40%) contrast(1.15);
		transition: transform 1s cubic-bezier(0.16, 1, 0.3, 1), filter 0.8s ease;
	}

	.split-image:hover img {
		transform: scale(1.03);
		filter: grayscale(0%) contrast(1.1);
	}

	.split-image-label {
		position: absolute;
		bottom: 1.5rem;
		left: 1.5rem;
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 0.6rem;
		font-weight: 400;
		letter-spacing: 0.3em;
		color: rgba(207, 204, 214, 0.3);
		text-transform: uppercase;
	}

	.split-text {
		display: flex;
		flex-direction: column;
		padding: 2rem;
		background: #00171f;
		position: relative;
    order: -1;
	}
	@media (min-width: 901px) {
		.split-text {
      order: 0;
    }
	}

	.split-text::before {
		content: '';
		position: absolute;
		top: 15%;
		left: 0;
		width: 1px;
		height: 70%;
		background: linear-gradient(180deg, transparent, #69140e, transparent);
		display: none;
	}
	@media (min-width: 901px) {
		.split-text::before { display: block; }
	}

	.split-headline {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: clamp(2.2rem, 11vw, 20rem);
		line-height: 0.95;
		color: #cfccd6;
		letter-spacing: 0.02em;
	}
	@media (min-width: 901px) {
		.split-headline {
			font-size: clamp(2.2rem, 3vw, 4rem);
		}
	}

	.split-body {
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 1.25rem;
		font-weight: 300;
		line-height: 1.85;
		color: rgba(207, 204, 214, 0.85);
		max-width: 400px;
	}
  .split-body-link {
    color: rgba(207, 204, 214, 0.85);
    text-decoration: underline;
    text-underline-offset: 0.25em;
    text-decoration-thickness: 1px;
    transition: color 0.3s ease;
  }

	/* ============================================
	   LYRIC SECTION
	   ============================================ */
	.lyric-section {
		width: 100%;
		padding: 4rem 0 0;
		background: #00171f;
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
		overflow: hidden;
	}
	.lyric-row {
		display: flex;
		flex-wrap: wrap;
		gap: 10px;
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		color: #cfccd6;
		line-height: 1;
		letter-spacing: 0.04em;
		font-size: clamp(2rem, 6vw, 8rem);
	}
	@media (min-width: 901px) {
		.lyric-row {
			font-size: clamp(3rem, 6vw, 8rem);
			gap: 0.4em;
		}
	}

	.lyric-row-0 {
		justify-content: flex-start;
	}
	.lyric-row-1 {
		justify-content: center;
	}
	.lyric-row-2 {
		justify-content: flex-end;
	}

	.lyric-word {
		display: inline-block;
		will-change: transform;
		transition: transform 0.1s linear;
	}
	.lw-accent {
		color: #a44200;
	}
	.lw-muted {
		color: rgba(207, 204, 214, 0.45);
	}
	.lw-dim {
		color: rgba(207, 204, 214, 0.25);
	}

	/* ============================================
	   RELEASES
	   ============================================ */
	.releases {
		padding: 4rem 2rem;
		max-width: 1500px;
		margin: 0 auto;
	}

	.releases-header {
		margin-bottom: 3rem;
	}

	.releases-title {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: clamp(3rem, 8vw, 7rem);
		line-height: 0.85;
		color: #cfccd6;
		letter-spacing: -0.01em;
	}

	.releases-grid {
		display: grid;
		grid-template-columns: 1fr;
		gap: 2.5rem;
		align-items: start;
	}
	@media (min-width: 901px) {
		.releases-grid {
			grid-template-columns: 1fr 1fr;
		}
	}

	.release-embed {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.release-embed iframe {
		border: 0;
		width: 100%;
		height: 352px;
		border-radius: 12px;
		cursor: auto;
	}

	.release-link {
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 1rem;
		font-weight: 400;
		letter-spacing: 0.35em;
		color: rgba(207, 204, 214, 0.85);
		text-transform: uppercase;
		text-decoration: none;
		transition: color 0.3s ease;
	}
	.release-link:hover {
		color: #a44200;
	}

	/* ============================================
	   WATCH
	   ============================================ */
	.watch-section {
		padding: 2rem 2rem;
		max-width: 1500px;
		margin: 0 auto;
	}
  @media (min-width: 901px) {
    .watch-section {
      padding: 4rem 2rem;
    }
  }

	.watch-header {
		margin-bottom: 3rem;
	}

	.watch-title {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: clamp(3rem, 8vw, 7rem);
		line-height: 0.85;
		color: #cfccd6;
		letter-spacing: -0.01em;
	}

	.watch-grid {
		display: grid;
		grid-template-columns: 1fr;
		gap: 2.5rem;
	}
	@media (min-width: 901px) {
		.watch-grid {
			grid-template-columns: 1fr 1fr;
		}
	}

	.watch-embed {
		width: 100%;
		overflow: hidden;
	}

	.watch-embed iframe {
		width: 100%;
		height: 350px;
		border: 0;
		cursor: auto;
	}
  @media (min-width: 901px) {
    .watch-embed iframe {
      height: 650px;
    }
  }

	/* ============================================
	   FIND US
	   ============================================ */
	.findus-section {
		padding: 2rem 2rem;
		max-width: 1500px;
		margin: 0 auto;
	}
  @media (min-width: 901px) {
    .findus-section {
      padding: 4rem 2rem;
    }
  }

	.findus-header {
		margin-bottom: 3rem;
	}

	.findus-title {
		font-family: 'Bebas Neue', 'Oswald', 'Anton', 'Impact', 'Arial Black', sans-serif;
		font-size: clamp(3rem, 8vw, 7rem);
		line-height: 0.85;
		color: #cfccd6;
		letter-spacing: -0.01em;
	}

	.findus-links {
		display: grid;
		grid-template-columns: 1fr;
		gap: 0;
		width: 100%;
	}
	@media (min-width: 601px) {
		.findus-links {
			grid-template-columns: repeat(3, 1fr);
		}
	}
	@media (min-width: 901px) {
		.findus-links {
			grid-template-columns: repeat(5, 1fr);
		}
	}

	.findus-links a {
		font-family: 'Barlow Condensed', sans-serif;
		font-size: 1rem;
		font-weight: 500;
		letter-spacing: 0.25em;
		color: rgba(207, 204, 214, 0.4);
		text-decoration: none;
		text-transform: uppercase;
		padding: 1.2rem 0;
		border-bottom: 1px solid rgba(207, 204, 214, 0.06);
		transition: all 0.35s ease;
		text-align: center;
	}

	.findus-links a:hover {
		color: #cfccd6;
		background: rgba(207, 204, 214, 0.03);
		border-color: #69140e;
	}

	/* ============================================
	   FULL WIDTH PHOTO
	   ============================================ */
	.fullwidth-photo {
		width: 100%;
		overflow: hidden;
    height: 600px;
    object-fit: cover;
    filter: grayscale(40%) contrast(1.15);
	}

	.fullwidth-photo img {
		width: 100%;
		height: 100%;
		display: block;
		object-fit: cover;
    filter: grayscale(20%) contrast(1.1) brightness(0.45);
	}

</style>
