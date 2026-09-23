<script>
	import { goto } from "$app/navigation";
	import { onMount } from "svelte";
	import BracketsColumn from "./BracketsColumn.svelte";

	export let leagueTeamManagers, players, brackets, selection, queryWeek;

	const { playoffsStart, champs, losers } = brackets;

	const champsBracket = champs.bracket;
	const losersBracket = losers.bracket;

	let bracket = [];
	let allMatches = [];

	let el, top;

	onMount(() => {
		if (queryWeek && queryWeek > 0 && queryWeek < playoffsStart) {
			goto(`/matchups?week=1`, { noscroll: true });
			selection = 'regular';
		} else {
			goto(`/matchups?week=${playoffsStart}`, { noscroll: true });
		}
	});

	const changeSelection = () => {
		if (selection == 'losers') {
			bracket = losersBracket;
		} else {
			bracket = champsBracket;
		}

		allMatches = [];

		for (const week of bracket) {
			allMatches = [...allMatches, ...week];
		}

		top = el?.getBoundingClientRect()?.top || 0;
		setSelected();
	};

	$: changeSelection(selection);

	let selected;

	const setSelected = () => {
		selected =
			bracket?.[0]
				?.filter(mp => !mp.bye)?.[0]?.[0]?.m
			|| null;
	};

	$: changeDisplay(selected);

	let matchup;
	let displayWeek;

	const changeDisplay = (s) => {
		if (!s) {
			matchup = null;
			return;
		}

		top = el?.getBoundingClientRect()?.top || 0;

		matchup =
			allMatches.find(match => match?.[0]?.m == s) || null;

		if (matchup) {
			displayWeek = playoffsStart + matchup[0].r - 1;
		}
	};
</script>

<style>
	.brackets {
		position: relative;
		width: 100%;
		margin: 0 auto;
		padding: 45px 25px 55px;
		box-sizing: border-box;
		font-family: 'Poppins', sans-serif;
		z-index: 5;
	}

	/*
		The actual playoff area.

		Everything is centered inside the white/cosmic
		container instead of being stretched across it.
	*/

	.main-bracket {
		position: relative;

		width: min(1100px, 100%);
		min-height: 760px;

		margin: 0 auto;

		display: flex;
		justify-content: center;
		align-items: flex-start;

		padding: 25px 10px 40px;

		box-sizing: border-box;

		border-radius: 30px;

		background:
			radial-gradient(
				circle at 50% 50%,
				rgba(255, 255, 255, 0.55),
				rgba(255, 255, 255, 0.15) 65%,
				rgba(221, 247, 255, 0.08)
			);

		border: 1px solid rgba(255, 255, 255, 0.55);

		box-shadow:
			0 15px 45px rgba(59, 40, 130, 0.08),
			inset 0 0 35px rgba(255, 255, 255, 0.35);

		overflow-x: auto;
		overflow-y: visible;
	}

	/*
		Each playoff round gets equal horizontal space.
	*/

	.round-column {
		position: relative;

		flex: 1 1 0;
		min-width: 300px;

		display: flex;
		justify-content: center;
		align-items: flex-start;

		box-sizing: border-box;

		padding-top: 0;
	}

	/*
		Semifinals sit halfway between the
		Quarterfinal matchups.
	*/

	.round-column.semifinals {
		padding-top: 97px;
	}

	/*
		Championship sits halfway between
		the two semifinal matchups.
	*/

	.round-column.championship {
		padding-top: 194px;
	}

	/*
		Extra breathing room around each column.

		This keeps the heading/title area from
		getting swallowed by the matchup card.
	*/

	.round-column :global(.bracketColumn) {
		position: relative;
		z-index: 2;
	}

	/*
		Make the round labels sit clearly above
		the matchup cards.
	*/

	.round-column :global(.label) {
		position: relative;
		z-index: 10;

		margin-bottom: 18px;

		padding: 4px 16px;

		border-radius: 999px;

		background: rgba(255, 255, 255, 0.78);

		border: 1px solid rgba(109, 40, 217, 0.18);

		box-shadow:
			0 5px 16px rgba(109, 40, 217, 0.08);

		color: #35116B;

		font-weight: 700;
	}

	/*
		Magical glow behind the playoff bracket.
	*/

	.main-bracket::before {
		content: '';

		position: absolute;

		top: 50%;
		left: 50%;

		transform: translate(-50%, -50%);

		width: 75%;
		height: 70%;

		background:
			radial-gradient(
				ellipse at center,
				rgba(124, 58, 237, 0.08),
				rgba(41, 182, 246, 0.05) 45%,
				transparent 72%
			);

		filter: blur(20px);

		pointer-events: none;

		z-index: 0;
	}

	/*
		Subtle magical stars/dots.
	*/

	.main-bracket::after {
		content: '✦     ✧        ✦          ✧      ✦';

		position: absolute;

		top: 12px;
		left: 50%;

		transform: translateX(-50%);

		width: 80%;

		text-align: center;

		color: rgba(109, 40, 217, 0.18);

		font-size: 15px;
		letter-spacing: 18px;

		pointer-events: none;

		z-index: 0;
	}

	.main-bracket::-webkit-scrollbar {
		height: 7px;
	}

	.main-bracket::-webkit-scrollbar-track {
		background: rgba(225, 237, 255, 0.55);
		border-radius: 10px;
	}

	.main-bracket::-webkit-scrollbar-thumb {
		background: linear-gradient(
			90deg,
			rgba(41, 182, 246, 0.45),
			rgba(109, 40, 217, 0.45)
		);

		border-radius: 10px;
	}

	@media (max-width: 1100px) {
		.main-bracket {
			width: 100%;
		}
	}

	@media (max-width: 900px) {
		.main-bracket {
			justify-content: flex-start;
			min-height: 700px;
		}

		.round-column {
			flex: 0 0 300px;
		}
	}

	@media (max-width: 700px) {
		.brackets {
			padding: 25px 10px 35px;
		}

		.main-bracket {
			padding-left: 5px;
			padding-right: 5px;
			border-radius: 20px;
		}

		.round-column {
			flex: 0 0 285px;
			min-width: 285px;
		}

		.round-column.semifinals {
			padding-top: 80px;
		}

		.round-column.championship {
			padding-top: 160px;
		}
	}
</style>

<div class="brackets" bind:this={el}>

	<div class="main-bracket">

		{#each bracket as matchCol, ix}

			<div
				class="round-column"
				class:semifinals={ix === 1}
				class:championship={ix === bracket.length - 1 && bracket.length >= 3}
			>

				<BracketsColumn
					bind:selected={selected}
					{leagueTeamManagers}
					{matchCol}
					{ix}
					{players}
					{playoffsStart}
					playoffLength={bracket.length}
					losers={selection == 'losers'}
				/>

			</div>

		{/each}

	</div>

</div>
