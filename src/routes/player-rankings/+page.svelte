<script>
	import { onMount } from 'svelte';
	import {
		getNflState,
		getLeagueRosters,
		getLeagueTeamManagers,
		getLeagueData,
		loadPlayers
	} from '$lib/utils/helper';
	import {
		getTeamFromTeamManagers
	} from '$lib/utils/helperFunctions/universalFunctions';

	const positions = [
		{ key: 'QB', label: 'QB' },
		{ key: 'RB', label: 'RB' },
		{ key: 'WR', label: 'WR' },
		{ key: 'TE', label: 'TE' },
		{ key: 'FLEX', label: 'FLEX' },
		{ key: 'DEF', label: 'DEF' },
		{ key: 'K', label: 'K' }
	];

	let activePosition = 'QB';
	let week = 1;
	let loading = true;
	let error = '';
	let rankings = [];
	let players = {};
	let rosterData = {};
	let teamManagers = null;
	let leagueData = null;
	let availableWeeks = [];
	let requestId = 0;

	function normalizeRosterData(data) {
		return data?.rosters || {};
	}

	function playerName(playerId) {
		const player = players?.[playerId];

		if (!player) return playerId;

		return (
			player.full_name ||
			player.fullName ||
			[
				player.fn || player.first_name || player.firstName || '',
				player.ln || player.last_name || player.lastName || ''
			]
				.filter(Boolean)
				.join(' ') ||
			player.name ||
			playerId
		);
	}

	function playerPosition(playerId) {
		const player = players?.[playerId];
		return String(player?.pos || player?.position || '').toUpperCase();
	}

	function playerPhoto(playerId) {
		const player = players?.[playerId];

		if (player?.pos === 'DEF' || player?.position === 'DEF') {
			return `https://sleepercdn.com/images/team_logos/nfl/${String(playerId).toLowerCase()}.png`;
		}

		return `https://sleepercdn.com/content/nfl/players/thumb/${playerId}.jpg`;
	}

	function teamNameForRoster(rosterId) {
		try {
			return (
				getTeamFromTeamManagers(
					teamManagers,
					String(rosterId),
					leagueData?.season
				)?.name ||
				`Team ${rosterId}`
			);
		} catch {
			return `Team ${rosterId}`;
		}
	}

	function nflTeam(playerId) {
		return players?.[playerId]?.team || '';
	}

	function buildRosterLookup() {
		const lookup = {};

		for (const [rosterId, roster] of Object.entries(rosterData)) {
			for (const playerId of roster?.players || []) {
				lookup[String(playerId)] = String(rosterId);
			}
		}

		return lookup;
	}

	async function fetchWeekMatchups(targetWeek) {
		const leagueId = leagueData?.league_id;

		if (!leagueId) return [];

		const response = await fetch(
			`https://api.sleeper.app/v1/league/${leagueId}/matchups/${targetWeek}`
		);

		if (!response.ok) {
			throw new Error(`Unable to load Week ${targetWeek} matchup data.`);
		}

		return response.json();
	}

	async function buildCumulativeStats(targetWeek) {
		const rosterLookup = buildRosterLookup();
		const totals = {};

		if (targetWeek < 1) return totals;

		const weeks = await Promise.all(
			Array.from({ length: targetWeek }, (_, index) =>
				fetchWeekMatchups(index + 1)
			)
		);

		weeks.forEach((matchups) => {
			for (const matchup of matchups || []) {
				for (const [playerId, rawPoints] of Object.entries(matchup?.players_points || {})) {
					const id = String(playerId);
					const points = Number(rawPoints) || 0;

					if (!totals[id]) {
						totals[id] = {
							playerId: id,
							points: 0,
							games: 0,
							rosterId: rosterLookup[id] || null
						};
					}

					totals[id].points += points;

					if (points !== 0) {
						totals[id].games += 1;
					}
				}
			}
		});

		return totals;
	}

	function eligible(item, position) {
		const pos = playerPosition(item.playerId);

		if (position === 'FLEX') {
			return ['RB', 'WR', 'TE'].includes(pos);
		}

		return pos === position;
	}

	function rankStats(totals, position) {
		return Object.values(totals)
			.filter((item) => players?.[item.playerId])
			.filter((item) => item.points > 0)
			.filter((item) => eligible(item, position))
			.map((item) => ({
				...item,
				name: playerName(item.playerId),
				position: playerPosition(item.playerId),
				nflTeam: nflTeam(item.playerId),
				team: item.rosterId ? teamNameForRoster(item.rosterId) : 'Free Agent',
				ppg: item.games > 0 ? item.points / item.games : item.points
			}))
			.sort((a, b) => {
				if (b.points !== a.points) return b.points - a.points;
				return b.ppg - a.ppg;
			})
			.slice(0, 10);
	}

	async function loadRankings(targetWeek) {
		const thisRequest = ++requestId;

		loading = true;
		error = '';

		try {
			const totals = await buildCumulativeStats(targetWeek);

			if (thisRequest !== requestId) return;

			const current = rankStats(totals, activePosition);

			if (targetWeek > 1) {
				const previousTotals = await buildCumulativeStats(targetWeek - 1);

				if (thisRequest !== requestId) return;

				const previous = rankStats(previousTotals, activePosition);
				const previousMap = new Map(
					previous.map((player, index) => [player.playerId, index + 1])
				);

				rankings = current.map((player, index) => ({
					...player,
					rank: index + 1,
					previousRank: previousMap.get(player.playerId) || null
				}));
			} else {
				rankings = current.map((player, index) => ({
					...player,
					rank: index + 1,
					previousRank: null
				}));
			}
		} catch (e) {
			if (thisRequest === requestId) {
				error = e?.message || 'Unable to load player rankings.';
				rankings = [];
			}
		} finally {
			if (thisRequest === requestId) {
				loading = false;
			}
		}
	}

	function changeWeek(direction) {
		const index = availableWeeks.indexOf(week);
		const nextIndex = index + direction;

		if (nextIndex < 0 || nextIndex >= availableWeeks.length) return;

		week = availableWeeks[nextIndex];
		loadRankings(week);
	}

	function selectPosition(position) {
		activePosition = position;
		loadRankings(week);
	}

	function movement(player) {
		if (!player.previousRank) return 'NEW';

		const change = player.previousRank - player.rank;

		if (change > 0) return `▲ +${change}`;
		if (change < 0) return `▼ ${Math.abs(change)}`;
		return '—';
	}

	function movementClass(player) {
		if (!player.previousRank) return 'new';

		const change = player.previousRank - player.rank;

		if (change > 0) return 'up';
		if (change < 0) return 'down';
		return 'steady';
	}

	onMount(async () => {
		try {
			const [nflState, rosters, managers, league, playerData] = await Promise.all([
				getNflState(),
				getLeagueRosters(),
				getLeagueTeamManagers(),
				getLeagueData(),
				loadPlayers(null)
			]);

			rosterData = normalizeRosterData(rosters);
			teamManagers = managers;
			leagueData = league;
			players = playerData?.players || {};

			const currentWeek = Number(nflState?.week) || 1;

			availableWeeks = Array.from(
				{ length: Math.max(currentWeek, 1) },
				(_, index) => index + 1
			);

			week = currentWeek;

			await loadRankings(week);
		} catch (e) {
			error = e?.message || 'Unable to load player rankings.';
			loading = false;
		}
	});
</script>

<svelte:head>
	<title>Player Rankings | Wizards Only</title>
</svelte:head>

<div class="rankings-page">
	<section class="header-card">
		<div>
			<div class="eyebrow">🏈 WIZARDS ONLY</div>
			<h1>PLAYER RANKINGS</h1>
			<p>League-specific rankings through Week {week}</p>
		</div>

		<div class="week-controls">
			<button
				disabled={availableWeeks.indexOf(week) <= 0}
				on:click={() => changeWeek(-1)}
			>
				←
			</button>

			<div class="week-pill">WEEK {week}</div>

			<button
				disabled={availableWeeks.indexOf(week) >= availableWeeks.length - 1}
				on:click={() => changeWeek(1)}
			>
				→
			</button>
		</div>
	</section>

	<div class="position-tabs">
		{#each positions as position}
			<button
				class:active={activePosition === position.key}
				on:click={() => selectPosition(position.key)}
			>
				{position.label}
			</button>
		{/each}
	</div>

	<section class="rankings-card">
		<div class="rankings-heading">
			<div>
				<h2>TOP 10 {activePosition === 'FLEX' ? 'FLEX' : activePosition + 'S'}</h2>
				<span>Season fantasy points • your league scoring</span>
			</div>

			<div class="legend">
				<span class="up">▲ Rising</span>
				<span class="down">▼ Falling</span>
			</div>
		</div>

		{#if loading}
			<div class="loading">✨ Loading the magic...</div>
		{:else if error}
			<div class="empty">
				<div class="empty-title">Couldn't load rankings</div>
				<div>{error}</div>
			</div>
		{:else if rankings.length}
			<div class="player-list">
				{#each rankings as player}
					<div class="player-row">
						<div class="rank-number">#{player.rank}</div>

						<div class="player-photo">
							<img
								src={playerPhoto(player.playerId)}
								alt={player.name}
								on:error={(event) => {
									event.currentTarget.src =
										'https://sleepercdn.com/images/v2/icons/player_default.webp';
								}}
							/>
						</div>

						<div class="player-main">
							<div class="player-name">{player.name}</div>

							<div class="player-meta">
								<span class="nfl-team">{player.nflTeam}</span>
								<span>•</span>
								<span>{player.team}</span>
							</div>
						</div>

						<div class="player-stats">
							<strong>{player.points.toFixed(1)}</strong>
							<span>PTS</span>
							<small>{player.ppg.toFixed(1)} PPG</small>
						</div>

						<div
							class:up={movementClass(player) === 'up'}
							class:down={movementClass(player) === 'down'}
							class:new={movementClass(player) === 'new'}
							class="movement"
						>
							{movement(player)}
						</div>
					</div>
				{/each}
			</div>
		{:else}
			<div class="empty">
				<div class="empty-title">No rankings yet</div>
				<div>Player statistics will appear once Sleeper has scored the week.</div>
			</div>
		{/if}
	</section>

	<div class="note">
		Rankings use the fantasy points Sleeper recorded for your league's matchups, so your custom league scoring is reflected.
		FLEX combines RB, WR and TE. DEF and K use their corresponding Sleeper position data.
	</div>
</div>

<style>
	.rankings-page {
		min-height: 100vh;
		padding: 18px 18px 50px;
		background: transparent;
	}

	.header-card,
	.rankings-card {
		max-width: 1200px;
		margin: 0 auto 16px;
		background: rgba(255, 255, 255, 0.96);
		border: 2px solid rgba(122, 80, 220, 0.22);
		border-radius: 22px;
		box-shadow: 0 8px 28px rgba(44, 69, 145, 0.16);
	}

	.header-card {
		min-height: 110px;
		padding: 20px 26px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 20px;
		border-top: 5px solid #5b189c;
	}

	.eyebrow {
		font-family: 'Luckiest Guy', cursive;
		color: #7a50dc;
		font-size: 14px;
		letter-spacing: 1px;
	}

	h1,
	h2 {
		font-family: 'Luckiest Guy', cursive;
		color: #5126b8;
		margin: 0;
		letter-spacing: 0.5px;
	}

	h1 {
		font-size: 34px;
		line-height: 1;
		margin-top: 4px;
	}

	.header-card p {
		margin: 7px 0 0;
		color: #6d5a94;
		font-size: 14px;
	}

	.week-controls {
		display: flex;
		align-items: center;
		gap: 8px;
	}

	.week-controls button {
		width: 42px;
		height: 42px;
		border-radius: 50%;
		border: 2px solid rgba(122, 80, 220, 0.25);
		background: white;
		color: #5126b8;
		font-size: 19px;
		font-weight: 800;
		cursor: pointer;
	}

	.week-controls button:disabled {
		opacity: 0.35;
		cursor: default;
	}

	.week-pill {
		background: #6124c9;
		color: white;
		padding: 12px 18px;
		border-radius: 999px;
		font-family: 'Luckiest Guy', cursive;
		white-space: nowrap;
	}

	.position-tabs {
		max-width: 1200px;
		margin: 0 auto 16px;
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		gap: 10px;
	}

	.position-tabs button {
		border: 2px solid rgba(122, 80, 220, 0.22);
		background: rgba(255, 255, 255, 0.94);
		color: #5126b8;
		border-radius: 15px;
		padding: 13px 10px;
		font-family: 'Luckiest Guy', cursive;
		font-size: 16px;
		cursor: pointer;
		transition: 0.15s ease;
	}

	.position-tabs button:hover {
		transform: translateY(-1px);
	}

	.position-tabs button.active {
		background: #6124c9;
		color: white;
		border-color: #6124c9;
		box-shadow: 0 6px 18px rgba(91, 24, 156, 0.22);
	}

	.rankings-card {
		padding: 22px;
	}

	.rankings-heading {
		display: flex;
		justify-content: space-between;
		align-items: end;
		gap: 15px;
		padding-bottom: 16px;
		border-bottom: 2px solid rgba(122, 80, 220, 0.16);
	}

	.rankings-heading h2 {
		font-size: 25px;
	}

	.rankings-heading span {
		color: #806da2;
		font-size: 12px;
	}

	.legend {
		display: flex;
		gap: 14px;
		font-size: 11px !important;
		font-weight: 700;
	}

	.legend .up {
		color: #149447;
	}

	.legend .down {
		color: #c33a4a;
	}

	.player-list {
		display: flex;
		flex-direction: column;
		gap: 9px;
		padding-top: 14px;
	}

	.player-row {
		min-height: 78px;
		display: grid;
		grid-template-columns: 55px 62px 1fr 105px 75px;
		align-items: center;
		gap: 14px;
		padding: 9px 12px;
		border: 1px solid rgba(122, 80, 220, 0.17);
		border-radius: 15px;
		background: linear-gradient(90deg, rgba(249, 247, 255, 0.96), rgba(238, 246, 255, 0.92));
	}

	.rank-number {
		font-family: 'Luckiest Guy', cursive;
		color: #5126b8;
		font-size: 21px;
		text-align: center;
	}

	.player-photo {
		width: 56px;
		height: 56px;
		border-radius: 50%;
		overflow: hidden;
		background: #e8e0fb;
		border: 3px solid rgba(122, 80, 220, 0.28);
	}

	.player-photo img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.player-name {
		font-family: 'Luckiest Guy', cursive;
		color: #3f2670;
		font-size: 20px;
	}

	.player-meta {
		display: flex;
		align-items: center;
		gap: 7px;
		margin-top: 3px;
		font-size: 11px;
		color: #8b78aa;
	}

	.nfl-team {
		font-weight: 800;
		color: #5e4a8b;
	}

	.player-stats {
		text-align: right;
	}

	.player-stats strong {
		display: block;
		font-family: 'Luckiest Guy', cursive;
		color: #5126b8;
		font-size: 22px;
		line-height: 1;
	}

	.player-stats span {
		display: block;
		font-size: 9px;
		font-weight: 800;
		letter-spacing: 1px;
		color: #8b78aa;
	}

	.player-stats small {
		display: block;
		margin-top: 3px;
		font-size: 9px;
		color: #7a50dc;
		font-weight: 700;
	}

	.movement {
		text-align: center;
		font-weight: 800;
		font-size: 13px;
	}

	.movement.up {
		color: #149447;
	}

	.movement.down {
		color: #c33a4a;
	}

	.movement.new {
		color: #7a50dc;
	}

	.movement.steady {
		color: #8b78aa;
	}

	.loading,
	.empty {
		padding: 55px 20px;
		text-align: center;
		color: #786696;
	}

	.empty-title {
		font-family: 'Luckiest Guy', cursive;
		color: #5126b8;
		font-size: 25px;
		margin-bottom: 6px;
	}

	.note {
		max-width: 1200px;
		margin: 0 auto;
		padding: 4px 8px;
		color: rgba(255, 255, 255, 0.88);
		font-size: 11px;
		text-align: center;
	}

	@media (max-width: 760px) {
		.header-card {
			align-items: flex-start;
			padding: 18px;
		}

		h1 {
			font-size: 28px;
		}

		.position-tabs {
			grid-template-columns: repeat(3, 1fr);
		}

		.player-row {
			grid-template-columns: 38px 48px 1fr 62px;
			gap: 8px;
		}

		.player-photo {
			width: 44px;
			height: 44px;
		}

		.player-name {
			font-size: 16px;
		}

		.player-meta {
			display: block;
			line-height: 1.35;
		}

		.player-stats {
			grid-column: 4;
			grid-row: 1;
		}

		.movement {
			grid-column: 4;
			grid-row: 2;
		}

		.rankings-heading {
			align-items: flex-start;
			flex-direction: column;
		}
	}
</style>
