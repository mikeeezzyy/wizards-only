<script>
	import LinearProgress from '@smui/linear-progress';
	import MatchupWeeks from './MatchupWeeks.svelte';
	import Brackets from './Brackets.svelte';
	import Button, { Group, Label } from '@smui/button';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { loadPlayers } from '$lib/utils/helper';

	export let queryWeek, leagueTeamManagersData, matchupsData, bracketsData, playersData;

	let players, matchupWeeks, year, week, regularSeasonLength, brackets, leagueTeamManagers;

	let loading = true;

	onMount(async () => {
		brackets = await bracketsData;

		const matchupsInfo = await matchupsData;

		leagueTeamManagers = await leagueTeamManagersData;

		matchupWeeks = matchupsInfo.matchupWeeks;
		year = matchupsInfo.year;
		week = matchupsInfo.week;
		regularSeasonLength = matchupsInfo.regularSeasonLength;

		const playersInfo = await playersData;

		players = playersInfo.players;

		loading = false;

		if (playersInfo.stale) {
			const newPlayersInfo = await loadPlayers(null, true);
			players = newPlayersInfo.players;
		}
	});

	const changeSelection = (s) => {
		if (s == 'regular') {
			queryWeek = 1;
			goto(`/matchups?week=1`, { noscroll: true });
		} else if (selection == 'regular') {
			queryWeek = 99;
			goto(`/matchups?week=99`, { noscroll: true });
		}

		selection = s;
	};

	let selection = 'regular';
</script>

<style>
	/* =====================================================
	   MAIN MATCHUPS CONTAINER
	===================================================== */

	.matchupsPage {
		position: relative;
		z-index: 2;

		width: min(94%, 1150px);

		margin: 35px auto 70px;

		padding: 35px 25px 50px;

		border-radius: 30px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.94),
				rgba(247, 251, 255, 0.88)
			);

		border: 1px solid rgba(92, 126, 190, 0.14);

		box-shadow:
			0 12px 40px rgba(37, 77, 150, 0.13);

		overflow: hidden;

		box-sizing: border-box;
	}


	/* Soft cloud-like decoration */

	.matchupsPage::before,
	.matchupsPage::after {
		content: '';

		position: absolute;

		border-radius: 50%;

		pointer-events: none;

		z-index: 0;

		filter: blur(2px);
	}

	.matchupsPage::before {
		width: 330px;
		height: 180px;

		top: -70px;
		right: -80px;

		background:
			radial-gradient(
				ellipse,
				rgba(214, 235, 255, 0.75) 0%,
				rgba(238, 228, 255, 0.45) 45%,
				transparent 75%
			);
	}

	.matchupsPage::after {
		width: 300px;
		height: 170px;

		bottom: -70px;
		left: -80px;

		background:
			radial-gradient(
				ellipse,
				rgba(220, 241, 255, 0.75) 0%,
				rgba(245, 230, 255, 0.4) 45%,
				transparent 75%
			);
	}


	/* =====================================================
	   CONTENT LAYER
	===================================================== */

	.content {
		position: relative;

		z-index: 2;
	}


	/* =====================================================
	   LOADING / MESSAGE
	===================================================== */

	.message {
		display: block;

		width: 85%;

		max-width: 500px;

		margin: 80px auto;

		padding: 28px 30px;

		box-sizing: border-box;

		border-radius: 22px;

		background:
			rgba(255, 255, 255, 0.9);

		border: 1px solid rgba(92, 126, 190, 0.14);

		box-shadow:
			0 8px 25px rgba(37, 77, 150, 0.10);

		color: #29416d;

		font-family: 'Poppins', sans-serif;

		text-align: center;
	}

	.message p {
		margin: 0 0 18px;

		color: #29416d;

		font-family: 'Poppins', sans-serif;

		font-weight: 600;
	}


	/* =====================================================
	   SEASON SELECTOR
	===================================================== */

	.buttonHolder {
		position: relative;

		z-index: 5;

		display: flex;

		flex-direction: column;

		align-items: center;

		gap: 10px;

		margin: 5px 0 28px;
	}


	/* Make both button groups feel like one polished control */

	:global(.buttonHolder .mdc-button-group) {
		border-radius: 14px;

		overflow: hidden;
	}


	:global(.selectionButtons) {
		min-height: 44px !important;

		padding: 0 22px !important;

		border-radius: 0 !important;

		font-family: 'Poppins', sans-serif !important;

		font-size: 0.78rem !important;

		font-weight: 700 !important;

		letter-spacing: 0.02em;

		text-transform: uppercase;

		color: #31558f !important;

		background:
			rgba(255, 255, 255, 0.9) !important;

		border-color:
			rgba(64, 111, 190, 0.2) !important;

		box-shadow: none !important;
	}


	/* Selected button */

	:global(.selectionButtons.mdc-button--raised) {
		color: white !important;

		background:
			linear-gradient(
				135deg,
				#6d28d9,
				#3b82f6
			) !important;

		border-color: transparent !important;

		box-shadow:
			0 4px 12px rgba(79, 70, 229, 0.25) !important;
	}


	:global(.selectionButtons:hover) {
		background:
			#eef6ff !important;

		color: #35116b !important;
	}


	:global(.selectionButtons.mdc-button--raised:hover) {
		color: white !important;

		background:
			linear-gradient(
				135deg,
				#7c3aed,
				#2563eb
			) !important;
	}


	/* =====================================================
	   PLAYOFF SUB-SELECTOR
	===================================================== */

	:global(.buttonHolder > .mdc-button-group:nth-child(2)) {
		margin-top: 2px;
	}


	:global(.buttonHolder > .mdc-button-group:nth-child(2) .selectionButtons) {
		min-height: 38px !important;

		font-size: 0.68rem !important;

		padding: 0 18px !important;
	}


	/* =====================================================
	   MATCHUP AREA
	===================================================== */

	:global(.matchupWeeks) {
		position: relative;

		z-index: 3;
	}


	/* =====================================================
	   BRACKET AREA
	===================================================== */

	:global(.brackets) {
		position: relative;

		z-index: 3;
	}


	/* =====================================================
	   MOBILE
	===================================================== */

	@media (max-width: 700px) {

		.matchupsPage {
			width: 96%;

			padding:
				25px
				10px
				40px;

			border-radius: 24px;
		}

		.buttonHolder {
			margin-bottom: 22px;
		}

		:global(.selectionButtons) {
			min-height: 40px !important;

			padding: 0 14px !important;

			font-size: 0.67rem !important;
		}

		:global(.buttonHolder > .mdc-button-group:nth-child(2) .selectionButtons) {
			padding: 0 11px !important;

			font-size: 0.6rem !important;
		}
	}


	@media (max-width: 480px) {

		.matchupsPage {
			width: 98%;

			padding:
				20px
				5px
				35px;

			border-radius: 20px;
		}

		:global(.selectionButtons) {
			padding: 0 10px !important;

			font-size: 0.6rem !important;
		}

		:global(.buttonHolder > .mdc-button-group:nth-child(2) .selectionButtons) {
			padding: 0 8px !important;

			font-size: 0.54rem !important;
		}
	}
</style>


{#if loading}

	<!-- Loading -->
	<div class="matchupsPage">

		<div class="content">

			<div class="message">

				<p>Loading league matchups...</p>

				<LinearProgress indeterminate />

			</div>

		</div>

	</div>

{:else}

	{#if matchupWeeks.length}

		<div class="matchupsPage">

			<div class="content">

				<!-- Season / Playoffs toggle -->
				<div class="buttonHolder">

					<Group variant="outlined">

						<!-- Regular Season -->
						<Button
							class="selectionButtons"
							onclick={() => changeSelection('regular')}
							variant={selection == 'regular' ? 'raised' : 'outlined'}
						>
							<Label>Regular Season</Label>
						</Button>

						<!-- Playoffs -->
						<Button
							class="selectionButtons"
							onclick={() => changeSelection('champions')}
							variant={
								selection == 'champions' || selection == 'losers'
									? 'raised'
									: 'outlined'
							}
						>
							<Label>Playoffs</Label>
						</Button>

					</Group>


					{#if selection == 'champions' || selection == 'losers'}

						<Group variant="outlined">

							<!-- Championship Bracket -->
							<Button
								class="selectionButtons"
								onclick={() => changeSelection('champions')}
								variant={selection == 'champions' ? 'raised' : 'outlined'}
							>
								<Label>Champions' Bracket</Label>
							</Button>


							<!-- Losers Bracket -->
							<Button
								class="selectionButtons"
								onclick={() => changeSelection('losers')}
								variant={selection == 'losers' ? 'raised' : 'outlined'}
							>
								<Label>Losers' Bracket</Label>
							</Button>

						</Group>

					{/if}

				</div>


				<!-- Regular season -->
				{#if selection == 'regular'}

					<MatchupWeeks
						{players}
						{queryWeek}
						{matchupWeeks}
						{regularSeasonLength}
						{year}
						{week}
						bind:selection={selection}
						{leagueTeamManagers}
					/>

				{/if}

			</div>

		</div>

	{:else}

		<div class="matchupsPage">

			<div class="content">

				<div class="message">

					<p>No upcoming matchups...</p>

				</div>

			</div>

		</div>

	{/if}


	<!-- Playoff brackets -->

	{#if brackets?.champs?.bracket?.[0]?.[0]?.[0]?.points &&
		(selection == 'champions' || selection == 'losers')}

		<div class="matchupsPage">

			<div class="content">

				<Brackets
					{queryWeek}
					{leagueTeamManagers}
					{players}
					{brackets}
					bind:selection={selection}
				/>

			</div>

		</div>

	{/if}

{/if}
