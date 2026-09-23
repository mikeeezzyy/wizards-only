<script>
	let showToc = true;

	function goToSection(id) {
		id?.scrollIntoView({ behavior: 'smooth', block: 'start' });
	}
</script>

<svelte:head>
	<title>Wizards Only Constitution</title>
	<meta
		name="description"
		content="The official Wizards Only Fantasy Football League Constitution and Rules."
	/>
</svelte:head>

<div class="constitution">

	<div class="hero">
		<div class="hero-kicker">WIZARDS ONLY</div>
		<h1>LEAGUE CONSTITUTION</h1>
		<p>The official rules, structure, scoring, finances, and league policies.</p>
	</div>

	<button class="tocToggle" on:click={() => showToc = !showToc}>
		<span>TABLE OF CONTENTS</span>
		<span>{showToc ? '−' : '+'}</span>
	</button>

	{#if showToc}
		<nav class="toc">
			<a href="#article-i">Article I — League Format</a>
			<a href="#article-ii">Article II — Keepers</a>
			<a href="#article-iii">Article III — Playoffs</a>
			<a href="#article-iv">Article IV — League Finances</a>
			<a href="#article-v">Article V — Waivers & Free Agency</a>
			<a href="#article-vi">Article VI — Trading</a>
			<a href="#article-vii">Article VII — Weekly Power Rankings</a>
			<a href="#article-viii">Article VIII — Custom Scoring</a>
			<a href="#article-ix">Article IX — League Punishment</a>
			<a href="#article-x">Article X — League Authority</a>
		</nav>
	{/if}

	<section id="article-i" bind:this={articleI}>
		<h2>Article I — League Format</h2>

		<h3>Section 1. League Overview</h3>
		<p>The Wizards Only Fantasy Football League shall consist of 12 teams and shall operate according to the rules and provisions contained in this Constitution.</p>

		<ul>
			<li><strong>League Size:</strong> 12 teams</li>
			<li><strong>Starting Lineup:</strong> QB, 2 RB, 2 WR, TE, 2 FLEX, Kicker, Defense/Special Teams</li>
			<li><strong>Bench:</strong> 7 spots</li>
			<li><strong>IR:</strong> 2 spots</li>
			<li><strong>Divisions:</strong> 4 divisions of 3 teams</li>
			<li><strong>Draft Format:</strong> All drafts shall be snake drafts.</li>
		</ul>

		<h3>Section 2. Divisions</h3>
		<p>The league shall be divided into four divisions, with three teams in each division. In future seasons, divisions will be redrafted by that year's division winners.</p>
	</section>

	<section id="article-ii">
		<h2>Article II — Keepers</h2>

		<h3>Section 1. Keeper Rules</h3>
		<ol>
			<li>Each team may keep <strong>3 players</strong> each season.</li>
			<li>There are <strong>no draft-round penalties</strong> for keepers.</li>
			<li>Kept players simply remain part of the team's roster entering the draft.</li>
			<li>Teams shall draft accordingly, taking their existing keepers into consideration.</li>
		</ol>
	</section>

	<section id="article-iii">
		<h2>Article III — Playoffs</h2>

		<h3>Section 1. Playoff Qualification</h3>
		<ol>
			<li>The <strong>4 division winners</strong> shall automatically qualify for the playoffs.</li>
			<li><strong>2 Wild Card teams</strong> shall also qualify, with one Wild Card selected from each conference based on the best remaining record.</li>
			<li>A total of <strong>6 teams</strong> shall make the playoffs.</li>
		</ol>

		<h3>Section 2. Playoff Seeding</h3>
		<p>Playoff seeding tiebreakers shall be determined by <strong>Total Points For (PF)</strong>.</p>
	</section>

	<section id="article-iv">
		<h2>Article IV — League Finances</h2>

		<h3>Section 1. Entry Fee</h3>
		<ol>
			<li>The league buy-in shall be <strong>$100 per team</strong>.</li>
			<li>Payment methods and deadlines shall be communicated before Week 1 kickoff.</li>
		</ol>

		<h3>Section 2. Payout Structure</h3>
		<table>
			<tbody>
				<tr><td>🥇 1st Place</td><td>$900</td></tr>
				<tr><td>🥈 2nd Place</td><td>$200</td></tr>
				<tr><td>🥉 3rd Place</td><td>$100</td></tr>
			</tbody>
		</table>
	</section>

	<section id="article-v">
		<h2>Article V — Waivers & Free Agency</h2>

		<h3>Section 1. FAAB — Free Agent Auction Budget</h3>
		<ol>
			<li>Waivers shall use <strong>FAAB</strong> instead of standard waiver priority.</li>
			<li>Each team shall receive a <strong>$100 season-long budget</strong>.</li>
			<li>Waiver claims shall be submitted as blind bids.</li>
			<li>The highest bid shall win the player.</li>
			<li>A team shall only spend FAAB when it successfully wins the player.</li>
			<li><strong>$0 bids are permitted.</strong></li>
		</ol>
	</section>

	<section id="article-vi">
		<h2>Article VI — Trading</h2>

		<h3>Section 1. Trade Eligibility</h3>
		<ol>
			<li>Future draft pick trading is permitted.</li>
			<li>Teams are responsible for considering the long-term consequences of trading future draft capital.</li>
		</ol>

		<h3>Section 2. Trade Vetoes</h3>
		<ol>
			<li>Trades may be vetoed by league vote.</li>
			<li><strong>8 of 12 votes (75%)</strong> are required to overturn a trade.</li>
			<li>The Commissioner may pause a questionable trade for review.</li>
			<li>The league retains final authority over whether a challenged trade is overturned.</li>
		</ol>
	</section>

	<section id="article-vii">
		<h2>Article VII — Weekly Power Rankings</h2>

		<p>Weekly Power Rankings shall be posted throughout the season.</p>

		<p>Rankings may consider:</p>
		<ul>
			<li>Team record</li>
			<li>Point totals</li>
			<li>Recent trends</li>
			<li>Overall team performance and league context</li>
		</ul>

		<p>Power Rankings may include commentary, subjective analysis, and league humor for entertainment purposes.</p>
	</section>

	<section id="article-viii">
		<h2>Article VIII — Custom Scoring</h2>

		<h3>Section 1. First Downs</h3>
		<ul>
			<li><strong>+0.5 points</strong> per rushing first down</li>
			<li><strong>+0.5 points</strong> per receiving first down</li>
		</ul>
		<p>These bonuses reward players who consistently move the chains.</p>

		<h3>Section 2. Return Yards & Touchdowns</h3>
		<ol>
			<li>If a manager rosters the individual returner (WR/RB/DB), the manager receives return yards and return touchdowns.</li>
			<li>If a manager only rosters the Defense/Special Teams, the manager receives return touchdowns only and does not receive return yards.</li>
		</ol>

		<h3>Section 3. Blocked Kicks</h3>
		<ul>
			<li>Blocked FG = <strong>3 points</strong></li>
			<li>Blocked PAT = <strong>3 points</strong></li>
		</ul>
	</section>

	<section id="article-ix">
		<h2>Article IX — League Punishment</h2>

		<h3>Section 1. Last-Place Punishment</h3>
		<ol>
			<li>The last-place team shall be subject to a league punishment.</li>
			<li>The specific punishment shall be determined by <strong>league vote before the season starts</strong>.</li>
			<li>The punishment must be completed by the deadline established by the league.</li>
			<li>Completion must be verified with <strong>photo or video proof</strong>.</li>
		</ol>

		<h3>Section 2. Failure to Complete Punishment</h3>
		<p>If the last-place team fails to complete the required punishment, the consequence shall be <strong>removal from the league</strong>.</p>
	</section>

	<section id="article-x">
		<h2>Article X — League Authority</h2>

		<p>The Wizards Only League shall operate according to this Constitution and the rules established herein. Where this Constitution grants authority to the league or Commissioner, that authority shall be exercised according to the provisions contained in this document.</p>

		<div class="closing">
			<div>🧙‍♂️</div>
			<strong>MAY THE BEST WIZARD WIN.</strong>
			<span>🏈</span>
		</div>
	</section>

</div>

<style>
	:global(html) {
		scroll-behavior: smooth;
	}

	.constitution {
		position: relative;
		z-index: 1;
		width: min(92%, 1000px);
		margin: 40px auto 80px;
		padding: 34px 46px 55px;
		background: linear-gradient(145deg, rgba(255,255,255,.97), rgba(241,247,255,.96));
		border: 1px solid rgba(91, 24, 156, .12);
		border-top: 5px solid #7a50dc;
		border-radius: 22px;
		box-shadow: 0 12px 34px rgba(44,69,145,.16);
		box-sizing: border-box;
	}

	.hero {
		text-align: center;
		padding: 10px 10px 26px;
	}

	.hero-kicker {
		font-family: 'Poppins', sans-serif;
		font-weight: 800;
		font-size: 12px;
		letter-spacing: 3px;
		color: #285fc4;
	}

	h1, h2 {
		font-family: 'Luckiest Guy', sans-serif;
		color: #35116b;
		letter-spacing: .5px;
	}

	h1 {
		font-size: 2.6rem;
		margin: 7px 0 8px;
	}

	.hero p {
		margin: 0;
		color: #61739a;
		font: 500 14px/1.6 'Poppins', sans-serif;
	}

	.tocToggle {
		width: 100%;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 14px 18px;
		margin: 8px 0 10px;
		border: 1px solid rgba(85,122,194,.18);
		border-radius: 14px;
		background: linear-gradient(135deg, rgba(236,244,255,.95), rgba(248,243,255,.95));
		color: #35116b;
		font: 800 14px 'Poppins', sans-serif;
		cursor: pointer;
	}

	.tocToggle span:last-child {
		font-size: 22px;
		color: #7a50dc;
	}

	.toc {
		display: grid;
		gap: 5px;
		padding: 12px 16px 20px;
		margin-bottom: 20px;
		border-bottom: 1px solid rgba(85,122,194,.16);
	}

	.toc a {
		color: #285fc4;
		text-decoration: none;
		font: 600 13px/1.5 'Poppins', sans-serif;
		padding: 4px 0;
	}

	.toc a:hover {
		color: #5b189c;
		transform: translateX(3px);
	}

	section {
		scroll-margin-top: 25px;
		padding-top: 8px;
	}

	section + section {
		margin-top: 42px;
		padding-top: 30px;
		border-top: 3px solid rgba(37,99,235,.12);
	}

	h2 {
		font-size: 1.75rem;
		margin: 0 0 18px;
	}

	h3 {
		font: 800 1.05rem/1.4 'Poppins', sans-serif;
		color: #253b70;
		margin: 24px 0 10px;
	}

	p, li {
		color: #273a62;
		font: 400 14px/1.75 'Poppins', sans-serif;
	}

	p {
		margin: 8px 0 14px;
	}

	ul, ol {
		margin: 8px 0 18px;
		padding-left: 26px;
	}

	li {
		margin: 4px 0;
	}

	strong {
		font-weight: 800;
		color: #334a7a;
	}

	table {
		width: 100%;
		border-collapse: separate;
		border-spacing: 0;
		overflow: hidden;
		border: 1px solid rgba(85,122,194,.15);
		border-radius: 14px;
		margin: 12px 0 20px;
		background: rgba(255,255,255,.7);
	}

	td {
		padding: 11px 14px;
		border-bottom: 1px solid rgba(85,122,194,.12);
		color: #273a62;
		font: 600 14px 'Poppins', sans-serif;
	}

	td:last-child {
		text-align: right;
		color: #35116b;
		font-weight: 800;
	}

	tr:last-child td {
		border-bottom: 0;
	}

	.closing {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 12px;
		margin-top: 38px;
		padding: 22px;
		border-radius: 16px;
		background: linear-gradient(135deg, rgba(232,242,255,.9), rgba(246,237,255,.9));
		border: 1px solid rgba(91,24,156,.12);
		color: #35116b;
		font: 800 14px 'Poppins', sans-serif;
	}

	.closing div,
	.closing span {
		font-size: 22px;
	}

	@media (max-width: 700px) {
		.constitution {
			width: calc(100% - 24px);
			margin: 20px auto 50px;
			padding: 24px 18px 36px;
			border-radius: 18px;
		}

		h1 {
			font-size: 2rem;
		}

		h2 {
			font-size: 1.45rem;
		}

		h3 {
			font-size: 1rem;
		}

		p, li, td {
			font-size: 13px;
		}

		.toc {
			padding-left: 8px;
			padding-right: 8px;
		}

		.closing {
			flex-wrap: wrap;
			text-align: center;
		}
	}
</style>
