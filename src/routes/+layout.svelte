<script>
	import { dev } from '$app/environment';
	import { injectAnalytics } from '@vercel/analytics/sveltekit';

	injectAnalytics({
		mode: dev ? 'development' : 'production'
	});

	const navItems = [
	  { label: 'Home', href: '/', icon: '🏈' },
	  { label: 'Matchups', href: '/matchups', icon: '🏈' },
	  { label: 'Rivalry', href: '/rivalry', icon: '🏈' },
	  { label: 'Power Rankings', href: '/power-rankings', icon: '🏈' },
	  { label: 'League Standings', href: '/standings', icon: '🏈' },
	  { label: 'League History', href: '/awards', icon: '🏈' },
	  { label: 'Records', href: '/records', icon: '🏈' },
	  { label: 'League Info', href: '/constitution', icon: '🏈' },
		{ label: 'Drafts', href: '/drafts', icon: '🏈' },
	  { label: 'Resources', href: '/resources', icon: '🏈' }
	];

	let mobileMenuOpen = false;
</script>

<svelte:head>
	<link rel="icon" type="image/png" href="/favicon.png" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

	<link
		href="https://fonts.googleapis.com/css2?family=Luckiest+Guy&family=Permanent+Marker&family=Poppins:wght@400;500;600;700;800&display=swap"
		rel="stylesheet"
	/>
</svelte:head>


<div class="site-shell">

	<!-- =========================
	     SIDEBAR
	========================== -->

	<aside class="sidebar">

		<div class="sidebar-top">

			<!-- BRAND -->
			<a href="/" class="brand">

				<div class="brand-title">
					WIZARDS
				</div>

				<div class="brand-title brand-only">
					ONLY
				</div>

				<div class="brand-subtitle">
					FANTASY FOOTBALL
				</div>

				<div class="brand-year">
					EST. 2025
				</div>

			</a>

			<!-- MOBILE MENU TOGGLE -->
			<button
				class="mobile-menu-toggle"
				class:open={mobileMenuOpen}
				type="button"
				aria-expanded={mobileMenuOpen}
				aria-label={mobileMenuOpen ? 'Close navigation menu' : 'Open navigation menu'}
				on:click={() => (mobileMenuOpen = !mobileMenuOpen)}
			>
				<span></span>
				<span></span>
				<span></span>
			</button>


			<!-- NAVIGATION -->
			<nav class="sidebar-nav" class:mobile-open={mobileMenuOpen}>

				{#each navItems as item}

					<a
						href={item.href}
						class="nav-item"
					>

						<span class="nav-icon">
							{item.icon}
						</span>

						<span class="nav-label">
							{item.label}
						</span>

					</a>

				{/each}

			</nav>

		</div>


		<!-- =========================
		     BOTTOM WIZARD
		========================== -->

		<div class="sidebar-wizard">

			<img
				src="/wizard.png"
				alt="Wizard"
			/>

		</div>

	</aside>


	<!-- =========================
	     MAIN CONTENT
	========================== -->

	<div class="main-content">

		<slot />

	</div>

</div>


<style>

	/* =================================
	   GLOBAL
	================================= */

	:global(html) {
		margin: 0;
		padding: 0;

		width: 100%;
		min-height: 100%;

		overflow-x: hidden;
	}

	:global(body) {
		margin: 0;
		padding: 0;

		width: 100%;
		min-height: 100%;

		overflow-x: hidden;

		font-family: 'Poppins', sans-serif;

		background-image:
			linear-gradient(
				rgba(244, 250, 255, 0.10),
				rgba(244, 250, 255, 0.10)
			),
			url('/wizards-background.png');

		background-size: cover;
		background-position: center;
		background-attachment: fixed;
		background-repeat: no-repeat;
	}

	:global(*) {
		box-sizing: border-box;
	}

	:global(a) {
		text-decoration: none;
	}


	/* =================================
	   SITE SHELL
	================================= */

	.site-shell {
		width: 100%;
		min-height: 100vh;

		display: flex;
	}


	/* =================================
	   SIDEBAR
	================================= */

	.sidebar {
		position: fixed;

		top: 0;
		left: 0;
		bottom: 0;

		z-index: 1000;

		width: 220px;

		background: #ffffff;

		display: flex;
		flex-direction: column;
		justify-content: space-between;

		padding: 16px 7px 8px;

		box-shadow:
			4px 0 20px rgba(40, 70, 150, 0.08);

		overflow: hidden;
	}


	/* =================================
	   BRAND
	================================= */

	.sidebar-top {
		width: 100%;
	}

	.brand {
		display: block;

		width: 100%;

		text-align: center;

		padding: 0 5px 17px;

		color: #2458bd;

		transition: transform 0.2s ease;
	}

	.brand:hover {
		transform: translateY(-1px);
	}


	.brand-title {
		font-family: 'Luckiest Guy', cursive;

		font-size: 29px;

		line-height: 0.88;

		letter-spacing: 1px;

		color: #285fc4;

		text-shadow:
			0 1px 0 rgba(255, 255, 255, 0.8);
	}

	.brand-only {
		margin-top: 4px;

		font-size: 26px;
	}


	.brand-subtitle {
		margin-top: 27px;

		font-family: 'Poppins', sans-serif;

		font-size: 10px;

		font-weight: 800;

		letter-spacing: 3px;

		color: #35569a;
	}


	.brand-year {
		margin-top: 27px;

		font-family: 'Poppins', sans-serif;

		font-size: 8px;

		font-weight: 700;

		letter-spacing: 3px;

		color: #7890c1;
	}


	/* =================================
	   MOBILE MENU TOGGLE
	================================= */

	.mobile-menu-toggle {
		display: none;

		border: 1px solid rgba(73, 126, 213, 0.18);
		background: linear-gradient(135deg, #edf7ff, #f8f3ff);
		border-radius: 14px;

		width: 46px;
		height: 40px;

		align-items: center;
		justify-content: center;
		flex-direction: column;
		gap: 4px;

		margin: 0 auto 10px;

		cursor: pointer;

		box-shadow: 0 3px 10px rgba(51, 93, 166, 0.08);
	}

	.mobile-menu-toggle span {
		display: block;
		width: 20px;
		height: 2px;
		border-radius: 2px;
		background: #31579b;
		transition: transform 0.2s ease, opacity 0.2s ease;
	}

	.mobile-menu-toggle.open span:nth-child(1) {
		transform: translateY(6px) rotate(45deg);
	}

	.mobile-menu-toggle.open span:nth-child(2) {
		opacity: 0;
	}

	.mobile-menu-toggle.open span:nth-child(3) {
		transform: translateY(-6px) rotate(-45deg);
	}


	/* =================================
	   NAVIGATION
	================================= */

	.sidebar-nav {
		width: 100%;

		display: flex;
		flex-direction: column;

		gap: 7px;

		margin-top: 14px;
	}


	.nav-item {
		position: relative;

		width: 100%;

		min-height: 47px;

		display: flex;
		align-items: center;

		padding: 6px 11px;

		border-radius: 18px;

		background:
			linear-gradient(
				135deg,
				rgba(241, 248, 255, 0.98),
				rgba(248, 250, 255, 0.98)
			);

		border: 1px solid rgba(73, 126, 213, 0.14);

		box-shadow:
			0 3px 10px rgba(51, 93, 166, 0.06);

		color: #29467b;

		transition:
			transform 0.18s ease,
			box-shadow 0.18s ease,
			background 0.18s ease,
			border-color 0.18s ease;
	}

	.nav-item:hover {
		transform: translateX(3px);

		background:
			linear-gradient(
				135deg,
				#edf7ff,
				#f8f3ff
			);

		border-color: rgba(83, 116, 210, 0.28);

		box-shadow:
			0 5px 15px rgba(52, 91, 166, 0.13);
	}

	.nav-item:active {
		transform: translateX(2px) scale(0.99);
	}


	/* ALL SIDEBAR ICONS ARE FOOTBALLS */

	.nav-icon {
		flex: 0 0 27px;

		width: 27px;
		height: 27px;

		display: flex;
		align-items: center;
		justify-content: center;

		font-family: sans-serif;

		font-size: 16px;

		line-height: 1;
	}


	/* SIDEBAR FONT */

	.nav-label {
		display: block;

		font-family: 'Luckiest Guy', cursive;

		font-size: 13px;

		line-height: 1;

		letter-spacing: 0.35px;

		color: #304a7a;

		white-space: nowrap;
	}


	/* =================================
	   WIZARD
	================================= */

	.sidebar-wizard {
		width: 100%;

		flex: 1;

		display: flex;

		align-items: flex-end;
		justify-content: center;

		padding-bottom: 0;

		pointer-events: none;
	}

	.sidebar-wizard img {
		display: block;

		/*
		 * Larger than before so the wizard
		 * fills the empty lower sidebar.
		 */
		width: 150px !important;
		max-width: 150px !important;

		height: auto;

		max-height: 220px;

		object-fit: contain;

		object-position: bottom center;
	}


	/* =================================
	   MAIN CONTENT
	================================= */

	.main-content {
		width: calc(100% - 220px);

		min-width: 0;
		min-height: 100vh;

		margin-left: 220px;

		position: relative;

		overflow-x: hidden;
	}


	/* =================================
	   REMOVE OLD LEAGUE PAGE NAVIGATION
	================================= */

	:global(.nav),
	:global(.navbar),
	:global(.navigation),
	:global(.top-nav),
	:global(.nav-container),
	:global(.navigation-container) {
		display: none !important;
	}


	/* =================================
	   REMOVE OLD FOOTER
	================================= */

	:global(footer),
	:global(.footer),
	:global(.site-footer),
	:global(.league-page-footer) {
		display: none !important;
	}


	/* =================================
	   TABLET
	================================= */

	@media (max-width: 1000px) {

		.sidebar {
			width: 190px;
		}

		.main-content {
			width: calc(100% - 190px);

			margin-left: 190px;
		}

		.brand-title {
			font-size: 27px;
		}

		.brand-only {
			font-size: 24px;
		}

		.nav-label {
			font-size: 12px;
		}

		.sidebar-wizard img {
			width: 135px !important;
			max-width: 135px !important;

			max-height: 200px;
		}
	}


	/* =================================
	   MOBILE
	================================= */

	@media (max-width: 700px) {

		.site-shell {
			display: block;
		}

		.sidebar {
			position: relative;

			width: 100%;

			height: auto;
			min-height: auto;

			padding: 16px 12px 12px;
		}

		.main-content {
			width: 100%;

			margin-left: 0;
		}

		.mobile-menu-toggle {
			display: flex;
		}

		.sidebar-nav {
			display: grid;

			grid-template-columns: 1fr 1fr;

			gap: 7px;

			max-height: 0;
			overflow: hidden;
			opacity: 0;
			margin-top: 0;
			transition: max-height 0.28s ease, opacity 0.2s ease, margin-top 0.28s ease;
		}

		.sidebar-nav.mobile-open {
			max-height: 700px;
			opacity: 1;
			margin-top: 7px;
		}

		.nav-item {
			min-height: 44px;

			padding: 6px 9px;
		}

		.nav-label {
			font-size: 12px;
		}

		.sidebar-wizard {
			display: none;
		}

		.brand {
			padding-bottom: 14px;
		}

		.brand-subtitle {
			margin-top: 18px;
		}

		.brand-year {
			margin-top: 15px;
		}
	}


	/* =================================
	   SMALL PHONES
	================================= */

	@media (max-width: 430px) {

		.sidebar-nav {
			grid-template-columns: 1fr;
		}

		.nav-label {
			font-size: 13px;
		}
	}

</style>
