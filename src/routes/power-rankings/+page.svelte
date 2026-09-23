<script>
	import { onMount } from 'svelte';
	import {
		getLeagueRosters,
		getLeagueTeamManagers,
		getLeagueData,
		loadPlayers
	} from '$lib/utils/helper';

	import {
		getTeamFromTeamManagers,
		getAvatarFromTeamManagers
	} from '$lib/utils/helperFunctions/universalFunctions';
	import { POWER_RANKINGS } from '$lib/PowerRankings/powerRankingsData';

	let loading = true;
	let error = '';
	let season = new Date().getFullYear();
	let week = 1;
	let selectedData = null;
	let rosterData = {};
	let teamManagers = null;
	let leagueData = null;
	let players = {};
	let weeklyTopPlayers = {};
	let weeklyTeamResults = {};


	function getAvailableWeeks() {
		return Object.keys(POWER_RANKINGS)
			.map(Number)
			.filter(Number.isFinite)
			.filter((week) => week >= 1)
			.sort((a, b) => a - b);
	}


	function getPowerRankingWeek(targetWeek) {
		return POWER_RANKINGS?.[targetWeek] || null;
	}


	function normalizeTeamName(value) {
		return String(value || '')
			.toLowerCase()
			.replace(/[^a-z0-9]/g, '');
	}


	function teamName(rosterId) {
		try {
			const direct = getTeamFromTeamManagers(
				teamManagers,
				rosterId,
				season
			)?.name;

			if (direct) return direct;

		} catch {
			// Fall through to the other Sleeper team-name sources.
		}


		const seasonMap =
			teamManagers?.teamManagersMap?.[season] ||
			teamManagers?.teamManagersMap?.[String(season)] ||
			{};


		const mapped =
			seasonMap?.[rosterId] ||
			seasonMap?.[String(rosterId)];


		if (mapped?.team?.name) {
			return mapped.team.name;
		}


		const roster = rosterData?.[rosterId];

		const ownerId = roster?.owner_id;


		const user =
			(teamManagers?.users || []).find(
				(item) =>
					String(item?.user_id) ===
					String(ownerId)
			);


		return (
			user?.metadata?.team_name ||
			user?.display_name ||
			''
		);
	}


	function findRosterId(ranking) {
		if (!ranking) return null;


		// If a manual ranking supplies a roster ID,
		// use it directly.
		const explicitRosterId =
			ranking.rosterId ??
			ranking.roster_id;


		if (
			explicitRosterId !== undefined &&
			explicitRosterId !== null
		) {
			const key =
				String(explicitRosterId);

			if (rosterData?.[key]) {
				return key;
			}
		}


		const target =
			normalizeTeamName(
				ranking.team
			);


		if (!target) return null;


		const entry =
			Object.entries(rosterData).find(
				([rosterId]) =>
					normalizeTeamName(
						teamName(rosterId)
					) === target
			);


		return entry?.[0] || null;
	}



	function teamAvatar(rosterId) {
		try {
			return (
				getAvatarFromTeamManagers(
					teamManagers,
					rosterId,
					season
				) || null
			);
		} catch {
			return null;
		}
	}

	/*
	 * Movement is calculated automatically from
	 * the most recent previously published week.
	 *
	 * Example:
	 *
	 * Week 1: Tax Evasion = #4
	 * Week 2: Tax Evasion = #1
	 *
	 * 4 - 1 = +3
	 *
	 * Therefore:
	 * ▲ +3
	 *
	 * We intentionally do NOT use previousRank
	 * from the data file anymore.
	 */
	function movementForTeam(team, targetWeek) {

		if (targetWeek <= 1) {
			return 0;
		}


		const currentData =
			getPowerRankingWeek(
				targetWeek
			);


		if (!currentData) {
			return 0;
		}


		const normalizedTeam =
			normalizeTeamName(team);


		const currentEntry =
			currentData.rankings?.find(
				(item) =>
					normalizeTeamName(
						item?.team
					) === normalizedTeam
			);


		if (!currentEntry) {
			return 0;
		}


		const priorWeeks =
			getAvailableWeeks()
				.filter(
					(value) =>
						value < targetWeek
				);


		const priorWeek =
			priorWeeks[
				priorWeeks.length - 1
			];


		if (!priorWeek) {
			return 0;
		}


		const priorData =
			getPowerRankingWeek(
				priorWeek
			);


		const priorEntry =
			priorData?.rankings?.find(
				(item) =>
					normalizeTeamName(
						item?.team
					) === normalizedTeam
			);


		if (!priorEntry) {
			return 0;
		}


		return (
			Number(priorEntry.rank) -
			Number(currentEntry.rank)
		);
	}


	function playerName(playerId) {

		const player =
			players?.[playerId];


		if (!player) {
			return playerId;
		}


		const firstName =
			player.fn ||
			player.first_name ||
			player.firstName ||
			'';


		const lastName =
			player.ln ||
			player.last_name ||
			player.lastName ||
			'';


		return (
			player.full_name ||
			player.fullName ||
			[
				firstName,
				lastName
			]
				.filter(Boolean)
				.join(' ') ||
			player.name ||
			playerId
		);
	}


	/*
	 * Sleeper player photos.
	 *
	 * Normal NFL players:
	 * https://sleepercdn.com/content/nfl/players/thumb/{player_id}.jpg
	 *
	 * Defense:
	 * Sleeper team logo.
	 */
	function playerPhoto(playerId) {

		const player =
			players?.[playerId];


		if (
			player?.pos === 'DEF' ||
			player?.position === 'DEF'
		) {
			return `https://sleepercdn.com/images/team_logos/nfl/${String(playerId).toLowerCase()}.png`;
		}


		return `https://sleepercdn.com/content/nfl/players/thumb/${playerId}.jpg`;
	}


	function playerFallbackPhoto(playerId) {

		const player =
			players?.[playerId];


		if (
			player?.pos === 'DEF' ||
			player?.position === 'DEF'
		) {
			return `https://sleepercdn.com/images/team_logos/nfl/${String(playerId).toLowerCase()}.png`;
		}


		return 'https://sleepercdn.com/images/v2/icons/player_default.webp';
	}


	function seasonPoints(rosterId) {

		const settings =
			rosterData?.[rosterId]?.settings;


		if (!settings) {
			return 0;
		}


		return (
			Number(settings.fpts) ||
			0
		) +
		(
			Number(
				settings.fpts_decimal
			) || 0
		) / 100;
	}


	async function fetchMatchups(targetWeek) {

		const response =
			await fetch(
				`https://api.sleeper.app/v1/league/${leagueData.league_id}/matchups/${targetWeek}`
			);


		if (!response.ok) {
			throw new Error(
				`Unable to load Week ${targetWeek} player stats.`
			);
		}


		return response.json();
	}


	function topThreeForRoster(
		matchups,
		rosterId
	) {

		const matchup =
			matchups?.find(
				(item) =>
					String(item?.roster_id) ===
					String(rosterId)
			);


		if (!matchup) {
			return [];
		}


		const points =
			matchup?.players_points || {};


		const rosterPlayers =
			matchup?.players ||
			rosterData?.[rosterId]?.players ||
			[];


		return rosterPlayers
			.map(
				(playerId) => ({
					playerId:
						String(playerId),

					points:
						Number(
							points?.[playerId]
						) || 0
				})
			)
			.filter(
				(player) =>
					player.points > 0
			)
			.sort(
				(a, b) =>
					b.points -
					a.points
			)
			.slice(0, 3);
	}


	function getStreak(
		rosterId,
		playerId,
		targetWeek,
		playerStats
	) {

		let streak = 0;


		for (
			let w = targetWeek;
			w >= 1;
			w--
		) {

			const weekPlayers =
				playerStats?.[w]?.[
					String(rosterId)
				] || [];


			if (
				weekPlayers.some(
					(player) =>
						String(
							player.playerId
						) ===
						String(playerId)
				)
			) {

				streak += 1;

			} else {

				break;

			}
		}


		return streak;
	}


	function buildKeyPlayers(
		rosterId,
		targetWeek,
		playerStats,
		playerData
	) {

		return (
			playerStats?.[
				targetWeek
			]?.[
				String(rosterId)
			] || []
		).map(
			(player) => ({
				...player,

				name:
					playerNameFromData(
						player.playerId,
						playerData
					),

				photo:
					playerPhotoFromData(
						player.playerId,
						playerData
					),

				fallbackPhoto:
					playerFallbackFromData(
						player.playerId,
						playerData
					),

				streak:
					getStreak(
						rosterId,
						player.playerId,
						targetWeek,
						playerStats
					)
			})
		);
	}


	function playerNameFromData(
		playerId,
		playerData
	) {

		const player =
			playerData?.[playerId];


		if (!player) {
			return playerId;
		}


		const firstName =
			player.fn ||
			player.first_name ||
			player.firstName ||
			'';


		const lastName =
			player.ln ||
			player.last_name ||
			player.lastName ||
			'';


		return (
			player.full_name ||
			player.fullName ||
			[
				firstName,
				lastName
			]
				.filter(Boolean)
				.join(' ') ||
			player.name ||
			playerId
		);
	}


	function playerPhotoFromData(
		playerId,
		playerData
	) {

		const player =
			playerData?.[playerId];


		if (
			player?.pos === 'DEF' ||
			player?.position === 'DEF'
		) {
			return `https://sleepercdn.com/images/team_logos/nfl/${String(playerId).toLowerCase()}.png`;
		}


		return `https://sleepercdn.com/content/nfl/players/thumb/${playerId}.jpg`;
	}


	function playerFallbackFromData(
		playerId,
		playerData
	) {

		const player =
			playerData?.[playerId];


		if (
			player?.pos === 'DEF' ||
			player?.position === 'DEF'
		) {
			return `https://sleepercdn.com/images/team_logos/nfl/${String(playerId).toLowerCase()}.png`;
		}


		return 'https://sleepercdn.com/images/v2/icons/player_default.webp';
	}



	function buildWeeklyTeamResults(matchups) {

		const results = {};
		const matchupGroups = {};

		for (const matchup of matchups || []) {

			const matchupId = matchup?.matchup_id;

			if (
				matchupId === undefined ||
				matchupId === null
			) {
				continue;
			}

			const key = String(matchupId);

			if (!matchupGroups[key]) {
				matchupGroups[key] = [];
			}

			matchupGroups[key].push(matchup);
		}


		for (const entries of Object.values(matchupGroups)) {

			if (entries.length < 2) {
				continue;
			}

			// Sleeper normally returns two roster entries per matchup.
			// Sort only for consistency; do not assume roster order.
			const first = entries[0];
			const second = entries[1];

			const firstPoints = Number(first?.points);
			const secondPoints = Number(second?.points);

			if (
				!Number.isFinite(firstPoints) ||
				!Number.isFinite(secondPoints)
			) {
				continue;
			}

			let firstResult = 'T';
			let secondResult = 'T';

			if (firstPoints > secondPoints) {
				firstResult = 'W';
				secondResult = 'L';
			} else if (firstPoints < secondPoints) {
				firstResult = 'L';
				secondResult = 'W';
			}

			results[String(first.roster_id)] = {
				points: firstPoints,
				opponentPoints: secondPoints,
				opponentRosterId: String(second.roster_id),
				result: firstResult
			};

			results[String(second.roster_id)] = {
				points: secondPoints,
				opponentPoints: firstPoints,
				opponentRosterId: String(first.roster_id),
				result: secondResult
			};
		}

		return results;
	}


	function getTeamGameResult(rosterId, targetWeek) {

		return (
			weeklyTeamResults?.[targetWeek]?.[
				String(rosterId)
			] || null
		);
	}


	function getTeamStreak(rosterId, targetWeek) {

		const current =
			getTeamGameResult(
				rosterId,
				targetWeek
			);

		if (!current) {
			return null;
		}

		const type = current.result;
		let count = 0;

		for (let w = targetWeek; w >= 1; w--) {

			const result =
				getTeamGameResult(
					rosterId,
					w
				);

			if (
				!result ||
				result.result !== type
			) {
				break;
			}

			count += 1;
		}

		return {
			type,
			count
		};
	}

	async function loadStats(targetWeek) {

		const requests = [];


		for (
			let w = 1;
			w <= targetWeek;
			w++
		) {

			requests.push(
				fetchMatchups(w)
			);

		}


		const results =
			await Promise.all(
				requests
			);


		const nextWeeklyTopPlayers =
			{};

		const nextWeeklyTeamResults =
			{};


		results.forEach(
			(matchups, index) => {

				const w =
					index + 1;


				nextWeeklyTopPlayers[w] =
					{};

				nextWeeklyTeamResults[w] =
					buildWeeklyTeamResults(
						matchups
					);


				for (
					const [
						rosterId
					]
					of Object.entries(
						rosterData
					)
				) {

					nextWeeklyTopPlayers[w][
						String(rosterId)
					] =
						topThreeForRoster(
							matchups,
							rosterId
						);
				}
			}
		);


		// Reassign the whole objects so
		// Svelte updates the cards immediately.
		weeklyTopPlayers =
			nextWeeklyTopPlayers;

		weeklyTeamResults =
			nextWeeklyTeamResults;
	}


	function applySelection() {

		selectedData =
			getPowerRankingWeek(
				week
			);
	}


	async function changeWeek(
		nextWeek
	) {

		const availableWeeks =
			getAvailableWeeks();


		if (
			!availableWeeks.includes(
				nextWeek
			)
		) {
			return;
		}


		week =
			nextWeek;


		applySelection();


		error = '';


		if (
			typeof window !==
			'undefined'
		) {

			const url =
				new URL(
					window.location.href
				);


			url.searchParams.set(
				'week',
				String(week)
			);


			window.history.replaceState(
				{},
				'',
				url
			);
		}


		try {

			await loadStats(
				week
			);

		} catch (e) {

			error =
				e?.message ||
				'Unable to load player stats.';
		}
	}


	onMount(
		async () => {

			try {

				const [
					rosters,
					managers,
					league,
					playerData
				] =
					await Promise.all([
						getLeagueRosters(),
						getLeagueTeamManagers(),
						getLeagueData(),
						loadPlayers(null)
					]);


				leagueData =
					league;


				season =
					Number(
						league?.season
					) ||
					new Date().getFullYear();


				rosterData =
					rosters?.rosters ||
					{};


				teamManagers =
					managers;


				players =
					playerData?.players ||
					{};


				// Refresh the player cache if
				// League Page says it is stale.
				if (
					playerData?.stale
				) {

					const freshPlayerData =
						await loadPlayers(
							null,
							true
						);


					players =
						freshPlayerData?.players ||
						players;
				}


				const availableWeeks =
					getAvailableWeeks();


				let requestedWeek =
					null;


				if (
					typeof window !==
					'undefined'
				) {

					const value =
						Number(
							new URLSearchParams(
								window.location.search
							).get('week')
						);


					if (
						Number.isFinite(
							value
						)
					) {

						requestedWeek =
							value;
					}
				}


				const currentLeagueWeek =
					Number(
						leagueData
							?.settings
							?.leg
					) || 0;


				const defaultWeek =
					requestedWeek ||
					(
						availableWeeks.includes(
							currentLeagueWeek
						)
							? currentLeagueWeek
							: availableWeeks[
									availableWeeks.length -
										1
								] || 1
					);


				week =
					availableWeeks.includes(
						defaultWeek
					)
						? defaultWeek
						: availableWeeks[0] ||
							1;


				selectedData =
					getPowerRankingWeek(
						week
					);


				if (
					typeof window !==
					'undefined'
				) {

					const url =
						new URL(
							window.location.href
						);


					url.searchParams.set(
						'week',
						String(week)
					);


					window.history.replaceState(
						{},
						'',
						url
					);
				}


				if (selectedData) {

					await loadStats(
						week
					);
				}


			} catch (e) {

				error =
					e?.message ||
					'Unable to load Power Rankings.';


			} finally {

				loading =
					false;

			}
		}
	);
</script>


{#if loading}

	<div class="loading-page">

		<div class="loading-card">
			✨ Loading the magic...
		</div>

	</div>


{:else if error}

	<div class="loading-page">

		<div class="loading-card">

			<h2>
				Something went wrong
			</h2>

			<p>
				{error}
			</p>

		</div>

	</div>


{:else if selectedData}

	<div class="power-page">

		<div class="controls-bar">

			<div class="controls-title">

				<span>
					🏈
				</span>

				POWER RANKINGS

			</div>


			<div class="week-controls">

				{#if getAvailableWeeks().indexOf(week) > 0}

					<button
						onclick={() =>
							changeWeek(
								getAvailableWeeks()[
									getAvailableWeeks().indexOf(week) -
										1
								]
							)
						}
					>
						← WEEK
						{getAvailableWeeks()[
							getAvailableWeeks().indexOf(week) - 1
						]}
					</button>

				{/if}


				<div class="current-week">
					WEEK {week}
				</div>


				{#if getAvailableWeeks().indexOf(week) < getAvailableWeeks().length - 1}

					<button
						onclick={() =>
							changeWeek(
								getAvailableWeeks()[
									getAvailableWeeks().indexOf(week) +
										1
								]
							)
						}
					>
						WEEK
						{getAvailableWeeks()[
							getAvailableWeeks().indexOf(week) + 1
						]}
						→
					</button>

				{/if}

			</div>

		</div>


		{#key week}
		<div class="rankings-grid">

			{#each selectedData.rankings as ranking, index}

				{@const rank =
					ranking.rank ||
					index + 1}


				{@const rosterId =
					findRosterId(
						ranking
					)}


				{@const keyPlayers =
					rosterId
						? buildKeyPlayers(
								rosterId,
								week,
								weeklyTopPlayers,
								players
							)
						: []}


				{@const teamAvatarUrl =
					rosterId
						? teamAvatar(rosterId)
						: null}

				{@const teamGameResult =
					rosterId
						? getTeamGameResult(
								rosterId,
								week
							)
						: null}

				{@const teamStreak =
					rosterId
						? getTeamStreak(
								rosterId,
								week
							)
						: null}

				<!-- IMPORTANT:
				     Movement is now based on
				     the previous published week,
				     NOT previousRank. -->
				{@const change =
					movementForTeam(
						ranking.team,
						week
					)}


				<article
					class="ranking-card"
				>


					<div
						class="wizard-frame"
					>

						<img
							src={`/wizard-${rank}.png`}
							alt={`Wizard rank ${rank}`}
						/>


						<div
							class="rank-badge"
						>
							#{rank}
						</div>

						{#if teamAvatarUrl}
							<img
								class="team-avatar"
								src={teamAvatarUrl}
								alt=""
							/>
						{/if}

					</div>


					<div
						class="ranking-content"
					>


						<div
							class="ranking-topline"
						>

							<div
								class="team-heading"
							>

								<h2>
									{ranking.team}
								</h2>


								{#if rosterId}

									<span
										class="record"
									>
										{rosterData[
											rosterId
										]?.settings
											?.wins ||
											0}
										-
										{rosterData[
											rosterId
										]?.settings
											?.losses ||
											0}
									</span>

								{/if}

							</div>


							<div
								class:up={
									change > 0
								}
								class:down={
									change < 0
								}
								class:steady={
									change === 0
								}
								class="movement"
							>

								{#if change > 0}

									▲ +{change}

								{:else if change < 0}

									▼ {Math.abs(
										change
									)}

								{:else}

									—

								{/if}

							</div>

						</div>


						<div
							class="commentary"
						>
							{ranking.commentary}
						</div>


						<div
							class="stat-grid"
						>

							<div
								class="stat-box"
							>

								<div
									class="stat-value"
								>
									{rosterId
										? seasonPoints(
												rosterId
											).toFixed(
												1
											)
										: '0.0'}
								</div>


								<div
									class="stat-label"
								>
									SEASON PF
								</div>

							</div>


							<div
								class="stat-box"
							>

								<div
									class="stat-value"
								>

									{rosterId
										? `${rosterData[
												rosterId
											]?.settings
												?.wins ||
												0}-${rosterData[
												rosterId
											]?.settings
												?.losses ||
												0}`
										: '0-0'}

								</div>


								<div
									class="stat-label"
								>
									RECORD
								</div>

							</div>

						</div>


						<div class="game-meta">

							<div class="game-meta-box">

								<div class="game-meta-value">
										{#if teamGameResult}
											{teamGameResult.points.toFixed(2)} - {teamGameResult.opponentPoints.toFixed(2)}
											<span
												class:win={teamGameResult.result === 'W'}
												class:loss={teamGameResult.result === 'L'}
												class:tie={teamGameResult.result === 'T'}
												class="game-result">
												({teamGameResult.result})
											</span>
										{:else}
											—
										{/if}
									</div>

									<div class="game-meta-label">
										WEEK {week} RESULT VS {teamGameResult?.opponentRosterId ? teamName(teamGameResult.opponentRosterId) : 'OPPONENT'}
									</div>

							</div>

							<div class="game-meta-box">

								<div class="game-meta-value">
									{#if teamStreak}
										{teamStreak.type}{teamStreak.count}
									{:else}
										—
									{/if}
								</div>

								<div class="game-meta-label">
									CURRENT STREAK
								</div>

							</div>

						</div>


						<div
							class="key-title"
						>
							TOP 3 PLAYERS
						</div>


						<div
							class="key-players"
						>

							{#if keyPlayers.length}

								{#each keyPlayers as player, playerIndex}

									<div
										class="key-player"
									>


										<span
											class="key-rank"
										>
											{playerIndex +
												1}
										</span>


										<div
											class="player-avatar"
										>

											<img
												src={player.photo}
												alt={player.name}
												onerror={(event) => {
													const img = event.currentTarget;

													if (
														img.dataset.fallback !==
														'true'
													) {
														img.dataset.fallback =
															'true';

														img.src =
															player.fallbackPhoto;
													}
												}}
											/>

										</div>


										<div
											class="player-info"
										>

											<div
												class="key-name"
											>
												{player.name}
											</div>


											<div
												class="key-points"
											>
												{player.points.toFixed(
													1
												)}
												pts
											</div>

										</div>


										<div
											class:hot={
												player.streak >=
												2
											}
											class="streak"
										>

											{player.streak}

											{#if player.streak >= 2}
												🔥
											{/if}

										</div>

									</div>

								{/each}


							{:else}

								<div
									class="no-stats"
								>
									Player stats will appear after the week is scored.
								</div>

							{/if}

						</div>

					</div>

				</article>

			{/each}

		</div>
		{/key}


		<div
			class="bottom-link"
		>

			<a href="/">
				← Back to Wizards Only Home
			</a>

		</div>

	</div>


{:else}

	<div
		class="loading-page"
	>

		<div
			class="loading-card"
		>
			No Power Rankings have been published for this week yet.
		</div>

	</div>

{/if}


<style>

	.power-page {
		min-height: 100vh;

		padding:
			18px
			18px
			50px;

		background: transparent;
	}


	.controls-bar {
		max-width: 1500px;

		margin:
			0 auto
			16px;

		min-height: 82px;

		padding:
			14px
			18px;

		background:
			rgba(
				255,
				255,
				255,
				0.96
			);

		border:
			2px solid
			rgba(
				122,
				80,
				220,
				0.22
			);

		border-radius: 22px;

		box-shadow:
			0 8px 24px
			rgba(
				44,
				69,
				145,
				0.12
			);

		display: flex;

		align-items: center;

		justify-content:
			space-between;

		gap: 18px;
	}


	.controls-title {
		font-family:
			'Luckiest Guy',
			cursive;

		font-size: 30px;

		color:
			#5126b8;

		letter-spacing:
			0.5px;

		white-space:
			nowrap;
	}


	.controls-title span {
		font-family: inherit;

		margin-right:
			7px;
	}


	.week-controls {
		display: flex;

		align-items: center;

		gap: 10px;
	}


	.week-controls button,
	.current-week {
		border:
			2px solid
			#dfcfff;

		background:
			#fff;

		color:
			#5126b8;

		border-radius:
			999px;

		padding:
			10px
			18px;

		font-family:
			'Luckiest Guy',
			cursive;

		letter-spacing:
			0.4px;

		font-size:
			14px;

		cursor:
			pointer;

		white-space:
			nowrap;
	}


	.week-controls button:disabled {
		opacity:
			0.35;

		cursor:
			not-allowed;
	}


	.current-week {
		background:
			#6d28d9;

		border-color:
			#6d28d9;

		color:
			#fff;

		box-shadow:
			0 5px 14px
			rgba(
				109,
				40,
				217,
				0.25
			);
	}


	.rankings-grid {
		max-width:
			1500px;

		margin:
			0 auto;

		display:
			grid;

		grid-template-columns:
			repeat(
				2,
				minmax(
					0,
					1fr
				)
			);

		gap:
			14px;
	}


	.ranking-card {
		display:
			grid;

		grid-template-columns:
			170px
			minmax(
				0,
				1fr
			);

		min-height:
			330px;

		overflow:
			hidden;

		border-radius:
			18px;

		border:
			2px solid
			rgba(
				122,
				80,
				220,
				0.25
			);

		background:
			#fff;

		box-shadow:
			0 8px 24px
			rgba(
				44,
				69,
				145,
				0.12
			);

		color:
			#38226b;
	}


	.wizard-frame {
		position:
			relative;

		min-height:
			330px;

		background:
			#eef3ff;

		overflow:
			hidden;
	}


	.wizard-frame img {
		width:
			100%;

		height:
			100%;

		min-height:
			330px;

		object-fit:
			cover;

		display:
			block;
	}


	.rank-badge {
		position:
			absolute;

		top:
			10px;

		left:
			10px;

		min-width:
			52px;

		height:
			44px;

		padding:
			0 8px;

		border-radius:
			10px;

		display:
			flex;

		align-items:
			center;

		justify-content:
			center;

		background:
			#fff;

		color:
			#2454b8;

		font-family:
			'Luckiest Guy',
			cursive;

		font-size:
			23px;

		box-shadow:
			0 4px 12px
			rgba(
				0,
				0,
				0,
				0.18
			);
	}


	.wizard-frame .team-avatar {
		position:
			absolute;

		left:
			50%;

		bottom:
			12px;

		transform:
			translateX(-50%);

		width:
			110px;

		height:
			110px;

		min-height:
			0;

		border-radius:
			50%;

		object-fit:
			cover;

		display:
			block;

		z-index:
			5;
	}



	.ranking-content {
		padding:
			16px
			16px
			14px;

		display:
			flex;

		flex-direction:
			column;

		min-width:
			0;
	}


	.ranking-topline {
		display:
			flex;

		align-items:
			flex-start;

		justify-content:
			space-between;

		gap:
			10px;

		border-bottom:
			1px solid
			#e3d8fa;

		padding-bottom:
			9px;
	}


	.team-heading h2 {
		margin:
			0;

		font-family:
			'Luckiest Guy',
			cursive;

		font-size:
			22px;

		line-height:
			1.05;

		letter-spacing:
			0.5px;

		color:
			#5126b8;
	}


	.record {
		display:
			block;

		margin-top:
			5px;

		font-size:
			13px;

		color:
			#7656a9;

		font-weight:
			800;
	}


	.movement {
		font-weight:
			900;

		font-size:
			17px;

		white-space:
			nowrap;
	}


	.movement.up {
		color:
			#149447;
	}


	.movement.down {
		color:
			#d83c55;
	}


	.movement.steady {
		color:
			#8067a9;
	}


	.commentary {
		margin:
			10px 0
			12px;

		color:
			#49366f;

		font-size:
			13px;

		line-height:
			1.45;

		min-height:
			42px;
	}


	.stat-grid {
		display:
			grid;

		grid-template-columns:
			repeat(
				2,
				minmax(
					0,
					1fr
				)
			);

		gap:
			8px;

		margin-bottom:
			12px;
	}


	.stat-box {
		border:
			2px solid
			#e3d8fa;

		border-radius:
			12px;

		padding:
			10px
			12px;

		background:
			#fff;
	}


	.stat-value {
		font-size:
			19px;

		font-weight:
			900;

		color:
			#3f286f;
	}


	.stat-label {
		margin-top:
			4px;

		font-size:
			9px;

		font-weight:
			900;

		letter-spacing:
			1px;

		color:
			#7656a9;
	}


	.game-meta {
		display:
			grid;

		grid-template-columns:
			repeat(
				2,
				minmax(
					0,
					1fr
				)
			);

		gap:
			8px;

		margin-bottom:
			10px;
	}


	.game-meta-box {
		border:
			2px solid
			#e3d8fa;

		border-radius:
			12px;

		padding:
			7px 10px;

		background:
			#fff;
	}


	.game-meta-value {
		font-size:
			16px;

		font-weight:
			900;

		color:
			#3f286f;

		white-space:
			nowrap;
	}


	.game-result {
		margin-left:
			4px;

		font-weight:
			1000;
	}


	.game-result.win {
		color:
			#149447;
	}


	.game-result.loss {
		color:
			#d83c55;
	}


	.game-result.tie {
		color:
			#8067a9;
	}


	.game-score {
		color:
			#8067a9;

		font-weight:
			800;
	}


	.game-meta-label {
		margin-top:
			2px;

		font-size:
			8px;

		font-weight:
			900;

		letter-spacing:
			0.9px;

		color:
			#7656a9;
	}


	.key-title {
		font-family:
			'Luckiest Guy',
			cursive;

		font-size:
			15px;

		color:
			#5126b8;

		letter-spacing:
			0.8px;

		margin-bottom:
			6px;
	}


	.key-players {
		display:
			flex;

		flex-direction:
			column;

		gap:
			5px;
	}


	.key-player {
		display:
			grid;

		grid-template-columns:
			24px
			32px
			minmax(
				0,
				1fr
			)
			42px;

		align-items:
			center;

		gap:
			7px;

		font-size:
			11px;
	}


	.key-rank {
		width:
			24px;

		height:
			24px;

		border-radius:
			7px;

		background:
			#6d28d9;

		color:
			#fff;

		display:
			flex;

		align-items:
			center;

		justify-content:
			center;

		font-weight:
			900;
	}


	.player-avatar {
		width:
			30px;

		height:
			30px;

		border-radius:
			50%;

		background:
			#eee4ff;

		display:
			flex;

		align-items:
			center;

		justify-content:
			center;

		overflow:
			hidden;

		font-size:
			14px;
	}


	.player-avatar img {
		width:
			100%;

		height:
			100%;

		object-fit:
			cover;

		display:
			block;
	}


	.player-info {
		min-width:
			0;
	}


	.key-name {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
	color: #38226b;
	font-weight: 800;
	line-height: 1.1;
}


	.key-points {
	margin-top: 0px;
	color: #8067a9;
	font-size: 10px;
	line-height: 1.1;
}


	.streak {
		text-align:
			right;

		color:
			#149447;

		font-weight:
			1000;

		font-size:
			13px;

		white-space:
			nowrap;
	}


	.streak.hot {
		color:
			#159447;
	}


	.no-stats {
		color:
			#8067a9;

		font-size:
			11px;
	}


	.bottom-link {
		max-width:
			1500px;

		margin:
			22px auto 0;

		text-align:
			center;
	}


	.bottom-link a {
		color:
			#5126b8;

		font-family:
			'Luckiest Guy',
			cursive;

		font-size:
			17px;

		text-decoration:
			none;
	}


	.bottom-link a:hover {
		text-decoration:
			underline;
	}


	.loading-page {
		min-height:
			100vh;

		display:
			flex;

		align-items:
			center;

		justify-content:
			center;

		padding:
			30px;
	}


	.loading-card {
		background:
			rgba(
				255,
				255,
				255,
				0.96
			);

		border-radius:
			20px;

		padding:
			30px
			40px;

		box-shadow:
			0 10px 30px
			rgba(
				30,
				64,
				175,
				0.16
			);

		color:
			#5126b8;

		text-align:
			center;
	}


	@media (max-width: 1000px) {

		.rankings-grid {
			grid-template-columns:
				1fr;

			max-width:
				760px;
		}
	}


	@media (max-width: 700px) {

		.controls-bar {
			flex-direction:
				column;

			align-items:
				stretch;
		}


		.controls-title {
			text-align:
				center;
		}


		.week-controls {
			justify-content:
				center;
		}
	}


	@media (max-width: 650px) {

		.power-page {
			padding:
				12px
				8px
				30px;
		}


		.ranking-card {
			grid-template-columns:
				115px
				minmax(
					0,
					1fr
				);
		}


		.wizard-frame,
		.wizard-frame img {
			min-height:
				280px;
		}


		.week-controls button,
		.current-week {
			font-size:
				11px;

			padding:
				8px
				10px;
		}


		.game-meta-value {
			font-size:
				14px;
		}


		.game-meta-label {
			font-size:
				7px;
		}


		.key-player {
			grid-template-columns:
				22px
				28px
				minmax(
					0,
					1fr
				)
				38px;

			gap:
				5px;
		}
	}

</style>
