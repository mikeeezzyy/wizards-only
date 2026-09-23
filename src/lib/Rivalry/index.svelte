<script>
	import Matchup from "$lib/Matchups/Matchup.svelte";
	import TradeTransaction from "$lib/Transactions/TradeTransaction.svelte";
	import {
		getLeagueRecords,
		getLeagueTransactions,
		getRivalryMatchups,
		loadPlayers,
		round
	} from "$lib/utils/helper";
	import {
		getRosterIDFromManagerIDAndYear
	} from "$lib/utils/helperFunctions/universalFunctions";
	import LinearProgress from '@smui/linear-progress';
	import { onMount } from "svelte";
	import ComparissonBar from "./ComparissonBar.svelte";
	import ManagerSelectors from "./ManagerSelectors.svelte";
	import RivalryControls from "./RivalryControls.svelte";

	export let leagueTeamManagers, playersInfo, transactionsInfo, recordsInfo, playerOne, playerTwo;

	/* =========================================
	   REFRESH STALE DATA
	========================================= */

	onMount(async () => {
		if (transactionsInfo.stale) {
			transactionsInfo = await getLeagueTransactions(false, true);
		}

		if (playersInfo.stale) {
			playersInfo = await loadPlayers(null, true);
		}

		if (recordsInfo.stale) {
			recordsInfo = await getLeagueRecords(true);
		}
	});


	/* =========================================
	   RIVALRY DATA
	========================================= */

	let rivalry = null;
	let loading = true;

	const analyzeRivalry = async (p1, p2) => {
		loading = true;
		matchup = null;

		if (p1 && p2) {
			rivalry = await getRivalryMatchups(p1, p2);
			loading = false;
		}
	};

	$: analyzeRivalry(playerOne, playerTwo);

	let selected = 0;

	$: matchup = rivalry?.matchups[selected]?.matchup;
	$: displayWeek = rivalry?.matchups[selected]?.week;
	$: year = rivalry?.matchups[selected]?.year;


	/* =========================================
	   TRADE HISTORY
	========================================= */

	const setTradeHistory = (p1, p2) => {
		if (!p1 || !p2) {
			return [];
		}

		const trades = transactionsInfo.transactions.filter(transaction => {
			if (transaction.type !== "trade") {
				return false;
			}

			const rosterIDOne = parseInt(
				getRosterIDFromManagerIDAndYear(
					leagueTeamManagers,
					playerOne,
					transaction.season
				)
			);

			const rosterIDTwo = parseInt(
				getRosterIDFromManagerIDAndYear(
					leagueTeamManagers,
					playerTwo,
					transaction.season
				)
			);

			if (rosterIDOne == rosterIDTwo) {
				return false;
			}

			return (
				transaction.rosters.includes(rosterIDOne) &&
				transaction.rosters.includes(rosterIDTwo)
			);
		});

		const move = (arr, from, to) => {
			arr.splice(to, 0, arr.splice(from, 1)[0]);
		};

		return trades.map(t => {
			const rosterIDOne = parseInt(
				getRosterIDFromManagerIDAndYear(
					leagueTeamManagers,
					playerOne,
					t.season
				)
			);

			const rosterIDTwo = parseInt(
				getRosterIDFromManagerIDAndYear(
					leagueTeamManagers,
					playerTwo,
					t.season
				)
			);

			const rosterOneStartLocation =
				t.rosters.indexOf(rosterIDOne);

			if (rosterOneStartLocation > 0) {
				move(
					t.rosters,
					rosterOneStartLocation,
					0
				);

				for (const tradeMove of t.moves) {
					move(
						tradeMove,
						rosterOneStartLocation,
						0
					);
				}
			}

			const rosterTwoStartLocation =
				t.rosters.indexOf(rosterIDTwo);

			const last = t.rosters.length - 1;

			if (rosterTwoStartLocation < last) {
				move(
					t.rosters,
					rosterTwoStartLocation,
					last
				);

				for (const tradeMove of t.moves) {
					move(
						tradeMove,
						rosterTwoStartLocation,
						last
					);
				}
			}

			return t;
		});
	};

	$: tradeHistory = setTradeHistory(
		playerOne,
		playerTwo
	);


	/* =========================================
	   PERFORMANCE DATA
	========================================= */

	const performanceOrderOne = [
		{
			field: "wins",
			label: "Wins",
			unit: "wins"
		},
		{
			field: "losses",
			label: "Losses",
			unit: "losses"
		},
		{
			field: "ties",
			label: "Ties",
			unit: "ties"
		}
	];

	const performanceOrderTwo = [
		{
			field: "fptsFor",
			label: "Fantasy Points For",
			unit: "fpts"
		},
		{
			field: "fptsAgainst",
			label: "Fantasy Points Against",
			unit: "fpts against"
		}
	];

	$: playerOneRecords =
		recordsInfo?.regularSeasonData?.leagueManagerRecords
			? recordsInfo.regularSeasonData.leagueManagerRecords[playerOne]
			: null;

	$: playerTwoRecords =
		recordsInfo?.regularSeasonData?.leagueManagerRecords
			? recordsInfo.regularSeasonData.leagueManagerRecords[playerTwo]
			: null;
</script>


<style>

	/* =========================================
	   PAGE
	========================================= */

	.rivalry-page {
		width: 100%;
		min-height: 100vh;

		padding: 28px 28px 60px;
	}


	/* =========================================
	   PAGE HEADER
	========================================= */

	.page-header {
		width: min(1100px, 100%);
		margin: 0 auto 18px;

		padding: 20px 26px;

		border-radius: 24px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.97),
				rgba(247, 251, 255, 0.94)
			);

		border: 1px solid rgba(73, 126, 213, 0.15);

		box-shadow:
			0 12px 32px rgba(30, 64, 175, 0.12);

		text-align: center;
	}


	.page-header h1 {
		margin: 0;

		font-family: 'Luckiest Guy', cursive;

		font-size: clamp(30px, 4vw, 46px);

		line-height: 1;

		letter-spacing: 1px;

		color: #35116B;
	}


	.page-header p {
		margin: 8px 0 0;

		font-family: 'Poppins', sans-serif;

		font-size: 13px;

		font-weight: 700;

		letter-spacing: 1.5px;

		text-transform: uppercase;

		color: #64748B;
	}


	/* =========================================
	   MANAGER SELECTOR
	========================================= */

	.rivalry-selection {
		width: min(1000px, 100%);

		margin: 0 auto 18px;

		padding: 18px 22px;

		border-radius: 22px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.98),
				rgba(248, 251, 255, 0.96)
			);

		border: 1px solid rgba(73, 126, 213, 0.15);

		box-shadow:
			0 8px 26px rgba(30, 64, 175, 0.10);
	}


	/* =========================================
	   MAIN CONTENT CARDS
	========================================= */

	.score-board {
		width: min(1000px, 100%);

		margin: 18px auto;

		padding: 22px 20px 28px;

		border-radius: 24px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.98),
				rgba(248, 251, 255, 0.95)
			);

		border: 1px solid rgba(73, 126, 213, 0.15);

		box-shadow:
			0 10px 30px rgba(30, 64, 175, 0.11);
	}


	/* =========================================
	   SECTION HEADINGS
	========================================= */

	h2,
	h3 {
		font-family: 'Luckiest Guy', cursive;
	}


	h3 {
		text-align: center;

		font-size: clamp(23px, 3vw, 30px);

		font-weight: 400;

		line-height: 1.1;

		margin: 4px 0 20px;

		color: #35116B;

		letter-spacing: 0.5px;
	}


	/* =========================================
	   LOADING
	========================================= */

	.loading {
		display: block;

		width: min(500px, 85%);

		margin: 70px auto;

		padding: 24px;

		text-align: center;

		border-radius: 20px;

		background: rgba(255, 255, 255, 0.95);

		box-shadow:
			0 10px 30px rgba(30, 64, 175, 0.10);

		color: #475569;
	}


	.loading p {
		margin: 0;

		font-family: 'Poppins', sans-serif;

		font-weight: 700;
	}


	/* =========================================
	   HELMET PLACEHOLDER
	========================================= */

	.helmets {
		display: block;

		width: min(650px, 85%);

		max-width: 800px;

		margin: 20px auto 40px;

		border-radius: 24px;

		filter:
			drop-shadow(
				0 15px 25px rgba(30, 64, 175, 0.14)
			);
	}


	/* =========================================
	   MATCHUP AREA
	========================================= */

	.matchup-wrapper {
		width: 100%;

		margin: 16px auto 4px;

		padding: 0;

		border-radius: 20px;
		overflow: hidden;

		background: rgba(255, 255, 255, 0.96);

		border: 1px solid rgba(73, 126, 213, 0.16);

		box-shadow: 0 8px 24px rgba(30, 64, 175, 0.08);
	}


	/* =========================================
	   TRADE HISTORY
	========================================= */

	.trades {
		width: min(750px, 95%);

		margin: 10px auto 0;
	}


	/* =========================================
	   EMPTY STATES
	========================================= */

	.empty-state {
		text-align: center;

		padding: 20px;

		border-radius: 16px;

		background: #f5f9ff;

		color: #64748B;

		font-family: 'Poppins', sans-serif;

		font-weight: 600;
	}


	/* =========================================
	   GLOBAL STYLING FOR ORIGINAL COMPONENTS
	========================================= */

	:global(.rivalrySelection) {
		width: 100%;
	}


	:global(.scoreBoard) {
		width: 100%;
		max-width: none;

		margin: 0;
		padding: 0;

		border: 0;

		background: transparent;

		box-shadow: none;
	}


	:global(.rivalrySelection select) {
		font-family: 'Poppins', sans-serif;
	}


	/* =========================================
	   RIVALRY MATCHUP SHELL
	========================================= */

	.matchup-wrapper :global(img) {
		border-radius: 12px;
	}

	.matchup-wrapper {
		--wizard-purple: #6d28d9;
		--wizard-blue: #29b6f6;
		--wizard-blue-light: #ddf7ff;
	}


	/* =========================================
	   MOBILE
	========================================= */

	@media (max-width: 700px) {

		.rivalry-page {
			padding: 16px 12px 40px;
		}

		.page-header {
			padding: 18px 16px;
			border-radius: 20px;
		}

		.rivalry-selection {
			padding: 14px 12px;
			border-radius: 18px;
		}

		.score-board {
			padding: 18px 10px 22px;
			border-radius: 20px;
		}

		h3 {
			margin-bottom: 16px;
		}
	}


	@media (max-width: 450px) {

		.rivalry-page {
			padding-left: 8px;
			padding-right: 8px;
		}

		.page-header h1 {
			font-size: 30px;
		}
	}

</style>


<div class="rivalry-page">

	<!-- =====================================
	     PAGE HEADER
	====================================== -->

	<div class="page-header">

		<h1>RIVALRY</h1>

		<p>
			Who has the magic?
		</p>

	</div>


	<!-- =====================================
	     MANAGER SELECTION
	====================================== -->

	<div class="rivalry-selection">

		<ManagerSelectors
			bind:playerOne={playerOne}
			bind:playerTwo={playerTwo}
			{leagueTeamManagers}
		/>

	</div>


	<!-- =====================================
	     LOADING / EMPTY STATE
	====================================== -->

	{#if loading}

		{#if playerOne && playerTwo}

			<div class="loading">

				<p>Analyzing rivalry...</p>

				<br />

				<LinearProgress indeterminate />

			</div>

		{:else}

			<div class="score-board">

				<div class="center">

					<img
						class="helmets"
						src="/helmets.png"
						alt="placeholder of helmets clashing"
					/>

				</div>

			</div>

		{/if}


	{:else}


		<!-- =====================================
		     HEAD TO HEAD + MATCHUPS
		====================================== -->

		{#if rivalry?.matchups.length > 0}

			<div class="score-board">

				<h3>
					Head to Head
				</h3>


				<ComparissonBar
					sideOne={rivalry.wins.one}
					sideTwo={rivalry.wins.two}
					label="Wins"
					unit="wins"
				/>


				<ComparissonBar
					sideOne={parseFloat(round(rivalry.points.one))}
					sideTwo={parseFloat(round(rivalry.points.two))}
					label="Points"
					unit="pts"
				/>


				<h3>
					Matchups
				</h3>


				<RivalryControls
					bind:selected={selected}
					{year}
					{displayWeek}
					length={rivalry.matchups.length}
				/>


				<div class="matchup-wrapper">

					<Matchup
						key={`${playerOne}-${playerTwo}`}
						ix={selected}
						active={selected}
						{year}
						{matchup}
						players={playersInfo.players}
						{displayWeek}
						expandOverride={true}
						{leagueTeamManagers}
					/>

				</div>

			</div>

		{/if}


		<!-- =====================================
		     TRADE HISTORY
		====================================== -->

		<div class="score-board">

			{#if playerOne && playerTwo}

				<h3>
					Trade History
				</h3>

				<div class="trades">

					{#each tradeHistory as transaction}

						<TradeTransaction
							players={playersInfo.players}
							{transaction}
							{leagueTeamManagers}
						/>

					{:else}

						<div class="empty-state">
							No trades yet...
						</div>

					{/each}

				</div>

			{/if}

		</div>


		<!-- =====================================
		     PERFORMANCE COMPARISON
		====================================== -->

		{#if playerOne && playerTwo && playerOneRecords && playerTwoRecords}

			<div class="score-board">

				<h3>
					Performance Comparison
				</h3>


				<ComparissonBar
					sideOne={parseFloat(round(
						playerOneRecords.wins /
						(
							playerOneRecords.wins +
							playerOneRecords.ties +
							playerOneRecords.losses
						) * 100
					))}
					sideTwo={parseFloat(round(
						playerTwoRecords.wins /
						(
							playerTwoRecords.wins +
							playerTwoRecords.ties +
							playerTwoRecords.losses
						) * 100
					))}
					label="Win Percentage"
					unit="%"
				/>


				{#each performanceOrderOne as stat}

					<ComparissonBar
						sideOne={parseFloat(
							round(playerOneRecords[stat.field])
						)}
						sideTwo={parseFloat(
							round(playerTwoRecords[stat.field])
						)}
						label={stat.label}
						unit={stat.unit}
					/>

				{/each}


				<ComparissonBar
					sideOne={parseFloat(round(
						playerOneRecords.fptsFor /
						(
							playerOneRecords.wins +
							playerOneRecords.ties +
							playerOneRecords.losses
						)
					))}
					sideTwo={parseFloat(round(
						playerTwoRecords.fptsFor /
						(
							playerTwoRecords.wins +
							playerTwoRecords.ties +
							playerTwoRecords.losses
						)
					))}
					label="Fantasy Points per Game"
					unit="fpts/game"
				/>


				{#each performanceOrderTwo as stat}

					<ComparissonBar
						sideOne={parseFloat(
							round(playerOneRecords[stat.field])
						)}
						sideTwo={parseFloat(
							round(playerTwoRecords[stat.field])
						)}
						label={stat.label}
						unit={stat.unit}
					/>

				{/each}


				<ComparissonBar
					sideOne={parseFloat(round(
						playerOneRecords.fptsFor /
						playerOneRecords.potentialPoints * 100
					))}
					sideTwo={parseFloat(round(
						playerTwoRecords.fptsFor /
						playerTwoRecords.potentialPoints * 100
					))}
					label="Lineup IQ"
					unit="%"
				/>

			</div>

		{/if}

	{/if}

</div>
