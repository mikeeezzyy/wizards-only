<script>
	import {
		getNflState,
		getLeagueRosters,
		getLeagueTeamManagers,
		getLeagueData
	} from '$lib/utils/helper';

	import {
		getTeamFromTeamManagers,
		getAvatarFromTeamManagers
	} from '$lib/utils/helperFunctions/universalFunctions';


	const dataPromise = Promise.all([
		getNflState(),
		getLeagueRosters(),
		getLeagueTeamManagers(),
		getLeagueData()
	]);


	function getTeamName(teamManagers, rosterId, year) {
		try {
			return (
				getTeamFromTeamManagers(
					teamManagers,
					rosterId,
					year
				)?.name ||
				`Team ${rosterId}`
			);
		} catch {
			return `Team ${rosterId}`;
		}
	}


	function getTeamAvatar(teamManagers, rosterId, year) {
		try {
			return (
				getAvatarFromTeamManagers(
					teamManagers,
					rosterId,
					year
				) || null
			);
		} catch {
			return null;
		}
	}


	function divisionName(division, leagueData) {
		const configuredName =
			leagueData?.metadata?.[
				`division_${division}`
			];

		if (configuredName) {
			return configuredName.toUpperCase();
		}

		const defaults = [
			'AFC WEST',
			'AFC SOUTH',
			'NFC NORTH',
			'NFC EAST'
		];

		return (
			defaults[
				Number(division) - 1
			] ||
			`DIVISION ${division}`
		);
	}


	function formatPoints(value) {
		return Number(value || 0).toFixed(1);
	}


	function getStandings(
		rosters,
		teamManagers,
		leagueData
	) {
		const year = leagueData?.season;

		const divisions =
			Number(
				leagueData?.settings?.divisions
			) || 1;

		const grouped = {};

		for (
			let i = 1;
			i <= divisions;
			i++
		) {
			grouped[i] = [];
		}


		for (
			const [rosterId, roster]
			of Object.entries(rosters || {})
		) {
			const division =
				Number(
					roster?.settings?.division
				) || 1;

			if (!grouped[division]) {
				grouped[division] = [];
			}


			const wins =
				Number(
					roster?.settings?.wins
				) || 0;

			const losses =
				Number(
					roster?.settings?.losses
				) || 0;

			const ties =
				Number(
					roster?.settings?.ties
				) || 0;

			const pointsFor =
				(
					Number(
						roster?.settings?.fpts
					) || 0
				) +
				(
					Number(
						roster?.settings?.fpts_decimal
					) || 0
				) / 100;


			grouped[division].push({
				rosterId,

				team: getTeamName(
					teamManagers,
					rosterId,
					year
				),

				avatar: getTeamAvatar(
					teamManagers,
					rosterId,
					year
				),

				wins,
				losses,
				ties,
				pointsFor
			});
		}


		for (
			const division
			of Object.keys(grouped)
		) {
			grouped[division].sort(
				(a, b) => {
					if (
						b.wins !== a.wins
					) {
						return (
							b.wins -
							a.wins
						);
					}

					if (
						a.losses !== b.losses
					) {
						return (
							a.losses -
							b.losses
						);
					}

					return (
						b.pointsFor -
						a.pointsFor
					);
				}
			);
		}

		return grouped;
	}
</script>


{#await dataPromise}

	<div class="loading-page">

		<div class="loading-cloud">
			<span>Loading the magic...</span>
		</div>

	</div>


{:then data}

	{@const nflState = data[0]}
	{@const rosterData = data[1]}
	{@const teamManagers = data[2]}
	{@const leagueData = data[3]}

	{@const rosters =
		rosterData?.rosters || {}}

	{@const currentWeek =
		Number(nflState?.week) || 1}

	{@const currentSeason =
		leagueData?.season}

	{@const standings =
		getStandings(
			rosters,
			teamManagers,
			leagueData
		)}


	<div class="standings-page">

		<div class="standings-card">

			<div class="card-heading">

				<div class="heading-left">

					<span class="heading-trophy">
						🏆
					</span>

					<h1>
						LEAGUE STANDINGS
					</h1>

				</div>

				<div class="heading-week">
					🏈 Week {currentWeek}
				</div>

			</div>


			<div class="season-bar">

				<span>
					{currentSeason} WIZARDS ONLY
				</span>

				<span>
					CURRENT STANDINGS
				</span>

			</div>


			<div class="division-grid">

				{#each Object.entries(standings) as [division, teams]}

					<div class="division-card">

						<div class="division-title">

							<span class="division-ball">
								🏈
							</span>

							<span>
								{divisionName(
									division,
									leagueData
								)}
							</span>

						</div>


						<div class="division-header">

							<span>#</span>
							<span>TEAM</span>
							<span>W-L</span>
							<span>PF</span>

						</div>


						{#each teams as team, index}

							<div class="standing-row">

								<span class="standing-rank">
									{index + 1}
								</span>


								<div class="standing-team">

									{#if team.avatar}

										<img
											src={team.avatar}
											alt=""
										/>

									{:else}

										<div class="mini-avatar">
											🏈
										</div>

									{/if}


									<span>
										{team.team}
									</span>

								</div>


								<span class="standing-record">
									{team.wins}-{team.losses}{#if team.ties}-{team.ties}{/if}
								</span>


								<span class="standing-points">
									{formatPoints(team.pointsFor)}
								</span>

							</div>

						{/each}

					</div>

				{/each}

			</div>

		</div>

	</div>


{:catch error}

	<div class="error-page">

		<div class="error-cloud">

			<h2>Something went wrong</h2>

			<p>
				{error?.message ||
					'Unable to load league standings.'}
			</p>

		</div>

	</div>

{/await}


<style>

	:global(html),
	:global(body) {
		min-height: 100%;
		margin: 0;
	}

	:global(body) {
		background-image: url('/wizards-background.png');
		background-size: cover;
		background-position: center top;
		background-repeat: no-repeat;
		background-attachment: fixed;
	}


	.standings-page {
		width: 100%;
		min-height: 100vh;

		padding: 28px 24px 60px;

		background: transparent;
	}


	.standings-card {
		position: relative;

		width: 100%;
		max-width: 1200px;

		margin: 0 auto;

    	min-height: calc(100vh - 75px);

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.96),
				rgba(240, 247, 255, 0.92)
			);

		border-radius: 20px;

		border-top: 5px solid #7a50dc;

		box-shadow:
			0 8px 28px
			rgba(44, 69, 145, 0.16);

		overflow: hidden;
	}


	.card-heading {
		display: flex;

		align-items: center;
		justify-content: space-between;

		min-height: 70px;

		padding: 12px 22px;

		border-bottom:
			1px solid
			rgba(80, 110, 180, 0.15);
	}


	.heading-left {
		display: flex;

		align-items: center;

		gap: 10px;
	}


	.heading-trophy {
		font-size: 24px;
	}


	h1 {
		margin: 0;

		font-family:
			'Luckiest Guy',
			cursive;

		font-size: 28px;

		line-height: 1;

		color: #2252bd;
	}


	.heading-week {
		padding: 7px 12px;

		border-radius: 12px;

		background:
			rgba(230, 241, 255, 0.9);

		font-size: 11px;

		font-weight: 800;

		color: #3265cc;
	}


	.season-bar {
		display: flex;

		align-items: center;
		justify-content: space-between;

		padding: 11px 22px;

		background:
			rgba(236, 245, 255, 0.65);

		border-bottom:
			1px solid
			rgba(80, 110, 180, 0.12);

		font-size: 9px;

		font-weight: 800;

		letter-spacing: 0.8px;

		color: #6274a7;
	}


  .division-grid {
	display: grid;
	grid-template-columns: 1fr 1fr;
	gap: 18px;
	padding: 18px;
}

.division-card {
	overflow: hidden;
	border: 1px solid rgba(85, 122, 194, 0.20);
	border-radius: 18px;
	background: rgba(247, 251, 255, 0.74);
	box-shadow: 0 3px 12px rgba(50, 85, 150, 0.05);
}

.division-title {
	display: flex;
	align-items: center;
	gap: 9px;
	padding: 16px 17px;
	font-family: 'Luckiest Guy', cursive;
	font-size: 21px;
	color: #5b189c;
	border-bottom: 1px solid rgba(85, 122, 194, 0.15);
}

.division-ball {
	font-size: 19px;
}

.division-header,
.standing-row {
	display: grid;
	grid-template-columns: 42px minmax(0, 1fr) 75px 75px;
	align-items: center;
}

.division-header {
	padding: 12px 16px;
	font-size: 10px;
	font-weight: 900;
	letter-spacing: 0.6px;
	color: #7182aa;
}

.standing-row {
	min-height: 76px;
	padding: 9px 16px;
	border-top: 1px solid rgba(85, 122, 194, 0.10);
	transition: background 0.15s ease;
}

.standing-rank {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 32px;
	height: 32px;
	border-radius: 10px;
	background: #eef4ff;
	font-size: 13px;
	font-weight: 900;
	color: #2857bd;
}

.standing-team {
	display: flex;
	align-items: center;
	min-width: 0;
	gap: 12px;
	font-size: 14px;
	font-weight: 700;
	color: #304678;
}

.standing-team img,
.mini-avatar {
	width: 48px;
	height: 48px;
	flex: 0 0 48px;
	border-radius: 50%;
	object-fit: cover;
}

.mini-avatar {
	display: flex;
	align-items: center;
	justify-content: center;
	background: #e8f1ff;
	font-size: 18px;
}

.standing-record {
	font-size: 13px;
	font-weight: 700;
	color: #49639c;
	text-align: center;
}

.standing-points {
	font-size: 13px;
	font-weight: 800;
	color: #2358c6;
	text-align: right;
}
	.loading-page,
	.error-page {
		display: flex;

		align-items: center;
		justify-content: center;

		min-height: 70vh;

		padding: 30px;
	}


	.loading-cloud,
	.error-cloud {
		padding: 30px 40px;

		border-radius: 30px;

		background:
			rgba(255, 255, 255, 0.94);

		box-shadow:
			0 8px 30px
			rgba(49, 72, 140, 0.15);

		text-align: center;
	}


	.loading-cloud span,
	.error-cloud h2 {
		margin: 0;

		font-family:
			'Luckiest Guy',
			cursive;

		color: #2855bd;
	}


	.error-cloud p {
		color: #64748b;
	}


	@media (max-width: 850px) {

		.division-grid {
			grid-template-columns: 1fr;
		}

	}


	@media (max-width: 600px) {

		.standings-page {
			padding: 16px 8px 30px;
		}


		.card-heading {
			padding: 10px 14px;

			min-height: 60px;
		}


		h1 {
			font-size: 21px;
		}


		.heading-trophy {
			font-size: 19px;
		}


		.heading-week {
			font-size: 9px;

			padding: 6px 8px;
		}


		.season-bar {
			padding: 9px 14px;

			font-size: 8px;
		}


		.division-grid {
			padding: 9px;

			gap: 9px;
		}


		.division-title {
			font-size: 14px;
		}


		.division-header,
		.standing-row {
			grid-template-columns:
				25px
				minmax(0, 1fr)
				45px
				48px;
		}


		.standing-row {
			min-height: 46px;

			padding: 5px 9px;
		}


		.standing-team {
			font-size: 10px;
		}


		.standing-team img,
		.mini-avatar {
			width: 28px;
			height: 28px;

			flex-basis: 28px;
		}

	}

</style>
