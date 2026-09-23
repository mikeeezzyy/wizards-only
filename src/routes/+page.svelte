<script>
	import {
		getNflState,
		getLeagueRosters,
		getLeagueTeamManagers,
		getLeagueData,
		loadPlayers,
		predictScores
	} from '$lib/utils/helper';

	import {
		renderManagerNames,
		getTeamFromTeamManagers,
		getAvatarFromTeamManagers
	} from '$lib/utils/helperFunctions/universalFunctions';

	import { POWER_RANKINGS } from '$lib/PowerRankings/powerRankingsData';


	async function getAllHistory() {
		try {
			const currentLeague = await getLeagueData();

			let leagueId =
				currentLeague?.status === 'complete'
					? currentLeague?.league_id
					: currentLeague?.previous_league_id;

			const seasons = [];

			while (leagueId) {
				const [leagueRes, winnersRes, losersRes] =
					await Promise.all([
						fetch(`https://api.sleeper.app/v1/league/${leagueId}`),
						fetch(`https://api.sleeper.app/v1/league/${leagueId}/winners_bracket`),
						fetch(`https://api.sleeper.app/v1/league/${leagueId}/losers_bracket`)
					]);

				if (
					!leagueRes.ok ||
					!winnersRes.ok ||
					!losersRes.ok
				) {
					break;
				}

				const [league, winners, losers] =
					await Promise.all([
						leagueRes.json(),
						winnersRes.json(),
						losersRes.json()
					]);

				const playoffRound =
					winners?.length
						? Math.max(
								...winners.map(
									(match) => Number(match?.r) || 0
								)
							)
						: 0;

				const toiletRound =
					losers?.length
						? Math.max(
								...losers.map(
									(match) => Number(match?.r) || 0
								)
							)
						: 0;

				const finalMatch =
					winners?.find(
						(match) =>
							Number(match?.p) === 1 &&
							match?.w
					) ||
					winners?.find(
						(match) =>
							Number(match?.r) === playoffRound &&
							match?.w
					);

				const toiletMatch =
					losers?.find(
						(match) =>
							Number(match?.r) === toiletRound &&
							match?.w
					);

				if (finalMatch?.w) {
					seasons.push({
						year: league?.season,
						champion: finalMatch.w,
						toilet: toiletMatch?.w || null
					});
				}

				leagueId =
					league?.previous_league_id || null;
			}

			return seasons;
		} catch (error) {
			console.error(
				'Unable to load league history:',
				error
			);

			return [];
		}
	}


	const dataPromise = Promise.all([
		getNflState(),
		getLeagueRosters(),
		getLeagueTeamManagers(),
		getLeagueData(),
		getAllHistory(),
		loadPlayers(null)
	]);


	function getTeamName(teamManagers, rosterId, year) {
		try {
			return getTeamFromTeamManagers(
				teamManagers,
				rosterId,
				year
			)?.name || `Team ${rosterId}`;
		} catch {
			return `Team ${rosterId}`;
		}
	}


	function getTeamAvatar(teamManagers, rosterId, year) {
		try {
			return getAvatarFromTeamManagers(
				teamManagers,
				rosterId,
				year
			);
		} catch {
			return null;
		}
	}


	function getManagerName(teamManagers, rosterId, year) {
		try {
			return renderManagerNames(
				teamManagers,
				rosterId,
				year
			) || '';
		} catch {
			return '';
		}
	}


	function getLoserTeam(teamManagers, season) {
		if (!season?.toilet) {
			return null;
		}

		try {
			return getTeamFromTeamManagers(
				teamManagers,
				season.toilet,
				season.year
			) || null;
		} catch {
			return null;
		}
	}


	function getLoserAvatar(teamManagers, season) {
		if (!season?.toilet) {
			return null;
		}

		try {
			return getAvatarFromTeamManagers(
				teamManagers,
				season.toilet,
				season.year
			) || null;
		} catch {
			return null;
		}
	}


	function normalizePowerRankings(scores) {

		if (!scores.length) {
			return [];
		}

		const maxScore = Math.max(
			...scores.map((team) => team.score)
		);

		return scores
			.map((team) => ({
				...team,

				power:
					maxScore > 0
						? (team.score / maxScore) * 100
						: 0
			}))
			.sort(
				(a, b) =>
					b.power - a.power
			);
	}


	function movementForTeam(team, targetWeek) {

		if (!targetWeek || targetWeek <= 1) {
			return 0;
		}

		const currentData =
			POWER_RANKINGS?.[targetWeek];

		if (!currentData) {
			return 0;
		}

		const currentEntry =
			currentData.rankings?.find(
				(item) => item.team === team
			);

		if (!currentEntry) {
			return 0;
		}

		const weeks =
			Object.keys(POWER_RANKINGS || {})
				.map(Number)
				.filter(Number.isFinite)
				.sort((a, b) => a - b);

		const priorWeek =
			[...weeks]
				.filter((value) => value < targetWeek)
				.pop();

		if (!priorWeek) {
			return 0;
		}

		const priorEntry =
			POWER_RANKINGS?.[priorWeek]?.rankings?.find(
				(item) => item.team === team
			);

		if (!priorEntry) {
			return 0;
		}

		return (
			Number(priorEntry.rank) -
			Number(currentEntry.rank)
		);
	}


	function getMovementLabel(change) {

		if (change > 0) {
			return `▲ +${change}`;
		}

		if (change < 0) {
			return `▼ ${Math.abs(change)}`;
		}

		return '—';
	}


</script>


{#await dataPromise}

	<div class="loading-page">

		<div class="loading-cloud">

			<span>
				Loading the magic...
			</span>

		</div>

	</div>


{:then data}

	{@const nflState = data[0]}

	{@const rosterData = data[1]}

	{@const teamManagers = data[2]}

	{@const leagueData = data[3]}

	{@const history = data[4]}

	{@const playerData = data[5]}

	{@const rosters =
		rosterData?.rosters || {}}

	{@const players =
		playerData?.players || {}}

	{@const currentSeason =
		leagueData?.season}

	{@const currentWeek =
		Number(nflState?.week) || 1}

	{@const playoffStart =
		Number(
			leagueData?.settings
				?.playoff_week_start
		) || 15}

	{@const regularSeasonEnd =
		Math.max(
			1,
			playoffStart - 1
		)}


	<!-- ============================================= -->
	<!-- POWER RANKINGS DATA                           -->
	<!-- ============================================= -->

	{@const powerRankingRaw =
		Object.entries(rosters).map(
			([rosterId, roster]) => {

				const rosterPlayers =
					(roster?.players || [])
						.map(
							(playerId) =>
								players[playerId]
						)
						.filter(Boolean);


				let score = 0;


				for (
					let week = currentWeek;
					week <= regularSeasonEnd;
					week++
				) {

					try {

						score +=
							Number(
								predictScores(
									rosterPlayers,
									week,
									leagueData
								)
							) || 0;

					} catch {
						// Ignore missing projection
						// data for an individual week.
					}
				}


				return {

					rosterId,

					team: getTeamName(
						teamManagers,
						rosterId,
						currentSeason
					),

					avatar: getTeamAvatar(
						teamManagers,
						rosterId,
						currentSeason
					),

					manager: getManagerName(
						teamManagers,
						rosterId,
						currentSeason
					),

					score

				};
			}
		)
	}


	{@const automaticPowerRankings =
		normalizePowerRankings(
			powerRankingRaw
		)}


	<!-- Find the most recent week that has actually
	     been published in Power Rankings data. -->

	{@const publishedWeeks =
		Object.keys(
			POWER_RANKINGS || {}
		)
			.map(Number)
			.filter(Number.isFinite)
			.sort(
				(a, b) => a - b
			)}


	{@const latestPublishedWeek =
		publishedWeeks.length
			? publishedWeeks[
					publishedWeeks.length - 1
				]
			: 1}


	{@const latestPublishedData =
		POWER_RANKINGS?.[
			latestPublishedWeek
		]}


	<!--
		Keep the existing automatic POWER score,
		but order the Home page according to the
		most recent manually published rankings.
	-->

	{@const powerRankings =
		latestPublishedData?.rankings?.map(
			(ranking, index) => {

				const normalizedTeam =
					String(
						ranking?.team || ''
					)
						.toLowerCase()
						.replace(
							/[^a-z0-9]/g,
							''
						);


				const automaticTeam =
					automaticPowerRankings.find(
						(team) =>
							String(
								team?.team || ''
							)
								.toLowerCase()
								.replace(
									/[^a-z0-9]/g,
									''
								) ===
							normalizedTeam
					);


				return {
					...(automaticTeam || {}),

					team:
						ranking?.team ||
						automaticTeam?.team ||
						`Team ${index + 1}`,

					rank:
						Number(
							ranking?.rank
						) ||
						index + 1,

					power:
						automaticTeam?.power ||
						0
				};
			}
		) || automaticPowerRankings }


	{@const sortedHistory =
		[...(history || [])].sort(
			(a, b) =>
				Number(b?.year || 0) -
				Number(a?.year || 0)
		)}


	<!-- ============================================= -->
	<!-- HOME PAGE                                      -->
	<!-- ============================================= -->

	<div class="home-page">


		<!-- ========================================= -->
		<!-- HERO                                       -->
		<!-- ========================================= -->

		<section class="hero">

			<div class="hero-week">
				🏈 Week {currentWeek}
			</div>

		</section>


		<!-- ========================================= -->
		<!-- DASHBOARD                                 -->
		<!-- ========================================= -->

		<div class="dashboard-grid">


			<!-- ======================================= -->
			<!-- POWER RANKINGS                           -->
			<!-- ======================================= -->

			<section
				class="dashboard-card rankings-card"
				id="power-rankings"
			>

				<div class="card-heading">

					<div class="heading-left">

						<span class="heading-ball">
							📈
						</span>

						<h2>
							POWER RANKINGS
						</h2>

					</div>


					<div class="heading-week">
						🏈 Week {latestPublishedWeek}
					</div>

				</div>


				<div class="rankings-header">

					<span>RANK</span>

					<span>TEAM</span>

					<span>MANAGER</span>

					<span>MOVE</span>

				</div>


				<div class="rankings-list">

					{#each powerRankings as team, index}

						{@const change =
							movementForTeam(
								team.team,
								latestPublishedWeek
							)}

						<div class="ranking-row">

							<div class="rank-number">
								{team.rank || index + 1}
							</div>

							<div class="ranking-team">

								{#if team.avatar}

									<img
										src={team.avatar}
										alt=""
									/>

								{:else}

									<div class="avatar-placeholder">
										🏈
									</div>

								{/if}

								<span>
									{team.team}
								</span>

							</div>

							<div class="ranking-manager">
								{team.manager}
							</div>

							<div
								class="ranking-movement"
								class:up={change > 0}
								class:down={change < 0}
								class:steady={change === 0}
							>
								{getMovementLabel(change)}
							</div>

						</div>
					{/each}

				</div>


				<div class="full-rankings-link">

					<a
						href={`/power-rankings?week=${latestPublishedWeek}`}
					>
						VIEW FULL POWER RANKINGS →
					</a>

				</div>

			</section>


			<!-- ======================================= -->
			<!-- HISTORY                                  -->
			<!-- ======================================= -->

			<div class="history-column">


				<!-- CHAMPION HISTORY -->

				<section
					class="dashboard-card history-card"
					id="league-history"
				>

					<div class="card-heading">

						<div class="heading-left">

							<span class="heading-crown">
								🏆
							</span>

							<h2>
								LEAGUE CHAMPION HISTORY
							</h2>

						</div>

					</div>


					{#if sortedHistory.length}

						{@const latestChampion =
							sortedHistory[0]}


						{@const latestChampionTeam =
							getTeamFromTeamManagers(
								teamManagers,
								latestChampion.champion,
								latestChampion.year
							)}


						{@const latestChampionAvatar =
							getAvatarFromTeamManagers(
								teamManagers,
								latestChampion.champion,
								latestChampion.year
							)}


						<div class="latest-history-card champion-card">

							<div class="champion-year">
								{latestChampion.year} Champion
							</div>


							{#if latestChampionAvatar}

								<img
									src={latestChampionAvatar}
									alt=""
								/>

							{:else}

								<div class="champion-placeholder">
									🏆
								</div>

							{/if}


							<div class="champion-name">
								{latestChampionTeam?.name || 'Champion'}
							</div>

						</div>


						{#if sortedHistory.length}

							<div class="history-rows">

								<div class="history-rows-title">
									CHAMPION HISTORY
								</div>


								{#each sortedHistory as season}

									{@const team =
										getTeamFromTeamManagers(
											teamManagers,
											season.champion,
											season.year
										)}


									{@const avatar =
										getAvatarFromTeamManagers(
											teamManagers,
											season.champion,
											season.year
										)}


									<div class="history-year-row">

										<span class="history-year">
											{season.year}
										</span>


										{#if avatar}

											<img
												class="history-mini-avatar"
												src={avatar}
												alt=""
											/>

										{:else}

											<span class="history-mini-placeholder">
												🏆
											</span>

										{/if}


										<span class="history-year-team">
											{team?.name || '—'}
										</span>

									</div>

								{/each}

							</div>

						{/if}


					{:else}

						<div class="no-history">
							No league champion history yet.
						</div>

					{/if}

				</section>


				<!-- LOSER HISTORY -->

				<section
					class="dashboard-card history-card loser-history-card"
					id="loser-history"
				>

					<div class="card-heading">

						<div class="heading-left">

							<span class="heading-crown">
								☠️
							</span>

							<h2>
								LEAGUE LOSER HISTORY
							</h2>

						</div>

					</div>


					{#if sortedHistory.length}

						{@const latestLoser =
							sortedHistory.find(
								(season) =>
									season?.toilet
							)}


						{#if latestLoser}

							{@const latestLoserTeam =
								getLoserTeam(
									teamManagers,
									latestLoser
								)}


							{@const latestLoserAvatar =
								getLoserAvatar(
									teamManagers,
									latestLoser
								)}


							<div class="latest-history-card champion-card loser-card">

								<div class="loser-year">
									{latestLoser.year} Loser
								</div>


								{#if latestLoserAvatar}

									<img
										src={latestLoserAvatar}
										alt=""
									/>

								{:else}

									<div class="champion-placeholder">
										☠
									</div>

								{/if}


								<div class="champion-name">
									{latestLoserTeam?.name || 'Loser'}
								</div>

							</div>


							{#if sortedHistory.some((season) => season?.toilet)}

								<div class="history-rows">

									<div class="history-rows-title">
										LOSER HISTORY
									</div>


									{#each sortedHistory.filter((season) => season?.toilet) as season}

										{@const team =
											getLoserTeam(
												teamManagers,
												season
											)}


										{@const avatar =
											getLoserAvatar(
												teamManagers,
												season
											)}


										<div class="history-year-row loser-history-row">

											<span class="history-year">
												{season.year}
											</span>


											{#if avatar}

												<img
													class="history-mini-avatar"
													src={avatar}
													alt=""
												/>

											{:else}

												<span class="history-mini-placeholder">
													☠
												</span>

											{/if}


											<span class="history-year-team">
												{team?.name || '—'}
											</span>

										</div>

									{/each}

								</div>

							{/if}


						{:else}

							<div class="no-history">
								No league loser history yet.
							</div>

						{/if}


					{:else}

						<div class="no-history">
							No league loser history yet.
						</div>

					{/if}

				</section>

			</div>

		</div>

	</div>


{:catch error}

	<div class="error-page">

		<div class="error-cloud">

			<h2>
				Something went wrong
			</h2>

			<p>
				{error?.message ||
					'Unable to load league data.'}
			</p>

		</div>

	</div>

{/await}


<style>

	/* ============================================= */
	/* PAGE                                           */
	/* ============================================= */

	.home-page {
		width: 100%;
		min-height: 100vh;

		padding: 0 14px 28px;

		background: transparent;
	}


	/* ============================================= */
	/* HERO                                           */
	/* ============================================= */

	.hero {
		position: relative;

		width: 100%;
		height: 325px;

		overflow: hidden;

		border-radius: 0 0 22px 22px;

		background-image:
			url('/wizards-banner.png');

		background-size: cover;

		background-position: center center;

		background-repeat: no-repeat;

		box-shadow:
			0 7px 20px rgba(47, 76, 157, 0.18);
	}


	.hero-week {
		position: absolute;

		right: 20px;
		bottom: 14px;

		padding: 7px 13px;

		border-radius: 14px;

		background: rgba(255, 255, 255, 0.92);

		font-size: 12px;
		font-weight: 700;

		color: #2455c5;

		box-shadow:
			0 3px 10px rgba(42, 68, 140, 0.12);
	}


	/* ============================================= */
	/* GRID                                           */
	/* ============================================= */

	.dashboard-grid {
		display: grid;

		grid-template-columns:
			minmax(0, 1.35fr)
			minmax(420px, 0.95fr);

		gap: 16px;

		margin-top: 16px;
	}


	/* ============================================= */
	/* CARD                                           */
	/* ============================================= */

	.dashboard-card {
		position: relative;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.96),
				rgba(240, 247, 255, 0.92)
			);

		border-radius: 18px;

		border-top: 4px solid #7a50dc;

		box-shadow:
			0 7px 22px rgba(44, 69, 145, 0.14);

		overflow: hidden;
	}

	/* Let the Power Rankings card use the full height
	   created by the history column, without enlarging
	   the player/team images. */
	.rankings-card {
		display: flex;
		flex-direction: column;
		min-height: 100%;
	}


	/* ============================================= */
	/* HEADINGS                                       */
	/* ============================================= */

	.card-heading {
		display: flex;

		align-items: center;
		justify-content: space-between;

		min-height: 58px;

		padding: 9px 16px;

		border-bottom:
			1px solid rgba(80, 110, 180, 0.15);
	}


	.heading-left {
		display: flex;

		align-items: center;

		gap: 9px;
	}


	.heading-left h2 {
		margin: 0;

		font-family: 'Luckiest Guy', cursive;

		font-size: 25px;
		line-height: 1;

		color: #2252bd;
	}


	.heading-ball,
	.heading-trophy,
	.heading-crown {
		font-size: 19px;
	}


	.heading-week {
		font-size: 11px;
		font-weight: 700;

		color: #3265cc;
	}


	/* ============================================= */
	/* POWER RANKINGS                                 */
	/* ============================================= */

	.rankings-header {
		display: grid;

		grid-template-columns:
			50px
			minmax(150px, 1fr)
			minmax(100px, 150px)
			60px;

		gap: 5px;

		padding: 11px 16px 8px;

		font-size: 10px;
		font-weight: 800;

		color: #6274a7;

		letter-spacing: 0.8px;
	}


	.rankings-list {
		display: flex;
		flex-direction: column;
		flex: 1;
		min-height: 0;
	}

	.ranking-row {
		display: grid;

		grid-template-columns:
			50px
			minmax(150px, 1fr)
			minmax(100px, 150px)
			60px;

		align-items: center;

		gap: 5px;

		flex: 1;
		min-height: 43px;

		padding: 5px 16px;

		border-top:
			1px solid rgba(81, 111, 180, 0.13);

		overflow: hidden;
	}


	.ranking-team {
		display: flex;

		align-items: center;

		gap: 9px;

		min-width: 0;

		height: 100%;
	}


	/* THIS is the important fix */
	.ranking-row .ranking-team img {
		width: 30px !important;
		height: 30px !important;

		min-width: 30px !important;
		min-height: 30px !important;

		max-width: 30px !important;
		max-height: 30px !important;

		flex: 0 0 30px;

		border-radius: 50%;

		object-fit: cover;

		display: block;
	}


	.ranking-team span {
		min-width: 0;

		overflow: hidden;

		text-overflow: ellipsis;

		white-space: nowrap;

		font-size: 13px;

		font-weight: 800;

		color: #263e78;
	}


	.ranking-manager {
		overflow: hidden;

		text-overflow: ellipsis;

		white-space: nowrap;

		font-size: 10px;

		color: #7590c4;
	}


	.ranking-movement {
		text-align: right;

		font-size: 13px;

		font-weight: 900;

		color: #8067a9;

		white-space: nowrap;
	}


	.ranking-movement.up {
		color: #149447;
	}


	.ranking-movement.down {
		color: #d83c55;
	}


	.ranking-movement.steady {
		color: #8067a9;
	}


	.rank-number {
		display: flex;

		align-items: center;

		justify-content: center;

		width: 30px;
		height: 30px;

		min-width: 30px;
		min-height: 30px;

		border-radius: 9px;

		background: #edf4ff;

		font-size: 12px;

		font-weight: 900;

		color: #2455c5;
	}


	.full-rankings-link {
		padding:
			10px 16px 14px;

		text-align: center;

		border-top:
			1px solid rgba(80, 110, 180, 0.12);
	}


	.full-rankings-link a {
		font-family:
			'Luckiest Guy',
			cursive;

		font-size: 12px;

		letter-spacing: 0.5px;

		color: #2455c5;

		text-decoration: none;
	}


	.full-rankings-link a:hover {
		text-decoration: underline;
	}


	/* ============================================= */
	/* HISTORY                                        */
	/* ============================================= */

	.history-column {
		grid-column: 2;

		display: flex;

		flex-direction: column;

		gap: 12px;

		align-self: start;

		min-width: 0;
	}


	.history-card,
	.loser-history-card {
		grid-column: auto;
		align-self: auto;
	}


	.latest-history-card {
		margin: 13px;

		padding: 18px;

		text-align: center;
	}


	.champion-card {
		border:
			1px solid rgba(80, 110, 180, 0.15);

		border-radius: 13px;

		background:
			rgba(255, 255, 255, 0.7);
	}


	.champion-year,
	.loser-year {
		margin-bottom: 9px;

		font-size: 11px;
		font-weight: 800;

		color: #2455c5;
	}


	.champion-card img,
	.champion-placeholder {
		width: 58px;
		height: 58px;

		margin: 0 auto 7px;

		border-radius: 50%;

		object-fit: cover;
	}


	.champion-placeholder {
		display: flex;

		align-items: center;
		justify-content: center;

		background: #fff;

		font-size: 25px;
	}


	.champion-name {
		font-size: 11px;
		font-weight: 800;

		color: #263e78;
	}


	.history-rows {
		margin: 0 13px 13px;

		border-top:
			1px solid rgba(80, 110, 180, 0.15);
	}


	.history-rows-title {
		padding: 10px 5px 6px;

		font-size: 9px;
		font-weight: 900;

		letter-spacing: 0.6px;

		color: #7585a8;
	}


	.history-year-row {
		display: grid;

		grid-template-columns:
			48px
			34px
			minmax(0, 1fr);

		align-items: center;

		gap: 8px;

		min-height: 48px;

		padding: 5px 8px;

		border-top:
			1px solid rgba(80, 110, 180, 0.11);
	}


	.history-year {
		font-size: 11px;
		font-weight: 900;

		color: #2455c5;
	}


	.history-mini-avatar,
	.history-mini-placeholder {
		width: 30px;
		height: 30px;

		border-radius: 50%;

		object-fit: cover;
	}


	.history-mini-placeholder {
		display: flex;

		align-items: center;
		justify-content: center;

		background: #fff;

		font-size: 16px;
	}


	.history-year-team {
		overflow: hidden;

		text-overflow: ellipsis;

		white-space: nowrap;

		font-size: 11px;
		font-weight: 800;

		color: #263e78;
	}


	.loser-card {
		border-color:
			rgba(109, 40, 217, 0.22);

		background:
			linear-gradient(
				135deg,
				#faf7ff,
				#f1eaff
			);
	}


	.loser-year {
		color: #6d28d9;
	}


	.loser-card .champion-name,
	.loser-history-row .history-year-team {
		color: #4f3975;
	}


	.no-history {
		padding: 25px;

		text-align: center;

		font-size: 12px;

		color: #7585a8;
	}


	/* ============================================= */
	/* MOBILE                                         */
	/* ============================================= */

	@media (max-width: 950px) {

		.dashboard-grid {
			grid-template-columns: 1fr;
		}


		.history-column {
			grid-column: auto;
		}


		.history-card,
		.loser-history-card {
			grid-column: auto;
		}
	}


	@media (max-width: 700px) {

		.home-page {
			padding: 0 8px 20px;
		}


		.hero {
			height: 245px;

			border-radius:
				0 0 18px 18px;
		}


		.dashboard-grid {
			gap: 12px;
		}


		.heading-left h2 {
			font-size: 20px;
		}


		.rankings-header,
		.ranking-row {
			grid-template-columns:
				38px
				minmax(120px, 1fr)
				80px
				50px;

			padding-left: 10px;
			padding-right: 10px;
		}


		.ranking-manager {
			display: none;
		}
	}


	@media (max-width: 450px) {

		.rankings-header {
			grid-template-columns:
				35px
				minmax(120px, 1fr)
				50px;
		}


		.ranking-row {
			grid-template-columns:
				35px
				minmax(120px, 1fr)
				50px;
		}


		.rankings-header span:nth-child(3),
		.ranking-manager {
			display: none;
		}


		.hero {
			height: 210px;
		}
	}

</style>
