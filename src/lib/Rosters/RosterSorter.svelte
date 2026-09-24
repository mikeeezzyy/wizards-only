<script>
	import Button, { Label } from '@smui/button';
	import Roster from './Roster.svelte';

	export let rosters, leagueTeamManagers, startersAndReserve, leagueData, players;

	const rosterPositions = leagueData.roster_positions;
	const numDivisions = leagueData.settings.divisions || 1;

	const divisions = [];

	for (let i = 0; i < numDivisions; i++) {
		divisions.push({
			name: leagueData.metadata ? leagueData.metadata[`division_${i + 1}`] : null,
			rosters: []
		});
	}

	for (const rosterID in rosters) {
		const roster = rosters[rosterID];
		const division =
			!roster.settings.division || roster.settings.division > numDivisions
				? 0
				: roster.settings.division - 1;

		divisions[division].rosters.push(roster);
	}

	let expanded = false;
</script>

<style>
	.rosters-page {
		width: 100%;
		max-width: 1500px;
		margin: 0 auto;
		padding: 18px 20px 50px;
		box-sizing: border-box;
	}

	.page-header {
		position: relative;
		overflow: hidden;
		text-align: center;
		margin: 0 auto 22px;
		padding: 22px 24px 20px;
		border-radius: 24px;
		background: rgba(255, 255, 255, 0.94);
		border: 1px solid rgba(73, 126, 213, 0.16);
		box-shadow: 0 10px 30px rgba(30, 64, 175, 0.10);
	}

	.page-header::before {
		content: "";
		position: absolute;
		inset: 0;
		background:
			radial-gradient(circle at 15% 20%, rgba(41, 182, 246, 0.13), transparent 28%),
			radial-gradient(circle at 85% 15%, rgba(124, 58, 237, 0.12), transparent 30%);
		pointer-events: none;
	}

	.page-header h1 {
		position: relative;
		margin: 0;
		font-family: "Luckiest Guy", "Permanent Marker", cursive;
		font-size: clamp(2rem, 4vw, 3rem);
		font-weight: 400;
		letter-spacing: 0.03em;
		color: var(--wizard-purple-dark, #35116B);
	}

	.page-header p {
		position: relative;
		margin: 5px 0 0;
		color: var(--wizard-muted, #64748B);
		font-weight: 600;
	}

	.expandButton {
		display: flex;
		justify-content: center;
		margin: 0 auto 24px;
	}

	.expandButton :global(button) {
		border-radius: 999px !important;
		border: 2px solid var(--wizard-blue, #29B6F6) !important;
		color: var(--wizard-purple-dark, #35116B) !important;
		background: rgba(255, 255, 255, 0.9) !important;
		font-weight: 800 !important;
		padding: 0 18px !important;
		box-shadow: 0 5px 15px rgba(30, 64, 175, 0.08);
	}

	.expandButton :global(button:hover) {
		background: var(--wizard-blue-light, #DDF7FF) !important;
	}

	.division-section {
		margin: 0 auto 30px;
	}

	.division-heading {
		position: relative;
		overflow: hidden;
		display: flex;
		align-items: center;
		gap: 12px;
		margin: 0 0 16px;
		padding: 16px 20px;
		border-radius: 18px;
		background: rgba(255, 255, 255, 0.94);
		border: 1px solid rgba(73, 126, 213, 0.16);
		box-shadow: 0 10px 30px rgba(30, 64, 175, 0.10);
	}

	.division-heading::before {
		content: "🏈";
		font-size: 1.15rem;
		position: relative;
		z-index: 1;
	}

	.division-heading::after {
		content: "";
		position: absolute;
		inset: 0;
		background:
			radial-gradient(circle at 15% 20%, rgba(41, 182, 246, 0.10), transparent 28%),
			radial-gradient(circle at 85% 15%, rgba(124, 58, 237, 0.09), transparent 30%);
		pointer-events: none;
	}

	.division-heading h2 {
		position: relative;
		z-index: 1;
		margin: 0;
		color: #35116B;
		font-family: "Luckiest Guy", "Permanent Marker", cursive;
		font-size: clamp(1.35rem, 2.4vw, 1.8rem);
		font-weight: 400;
		letter-spacing: 0.04em;
	}

	.division {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		align-items: start;
		gap: 20px;
		width: 100%;
		margin: 0 auto;
	}

	@media (max-width: 1100px) {
		.division {
			grid-template-columns: repeat(2, minmax(0, 1fr));
		}
	}

	@media (max-width: 700px) {
		.rosters-page {
			padding: 12px 10px 40px;
		}

		.page-header {
			padding: 18px 16px;
			border-radius: 20px;
		}

		.division {
			grid-template-columns: 1fr;
			gap: 14px;
		}
	}
</style>

<div class="rosters-page">
	<section class="page-header">
		<h1>ROSTERS</h1>
		<p>Every wizard. Every starter. Every bench.</p>
	</section>

	<div class="expandButton">
		<Button onclick={() => { expanded = !expanded }} variant="outlined">
			<Label>{expanded ? "Minimize" : "Expand"} All Benches</Label>
		</Button>
	</div>

	{#each divisions as division, ix}
		<section class="division-section">
			{#if division.name}
				<div class="division-heading">
					<h2>{division.name}</h2>
				</div>
			{/if}

			<div class="division">
				{#each division.rosters as roster}
					<Roster
						division={ix + 1}
						{expanded}
						{rosterPositions}
						{roster}
						{leagueTeamManagers}
						{players}
						{startersAndReserve}
					/>
				{/each}
			</div>
		</section>
	{/each}
</div>
