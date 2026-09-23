<script>
	import { Awards } from '$lib/components';
	import { waitForAll } from '$lib/utils/helper';
	import LinearProgress from '@smui/linear-progress';

	export let data;

	const { awardsData, teamManagersData } = data;

	let selectedYear = null;


	function getAvailableYears(podiums, teamManagers) {
		const years = new Set();

		// Completed seasons from awards data
		for (const podium of podiums || []) {
			const year = Number(podium?.year);

			if (year) {
				years.add(year);
			}
		}

		// Seasons available through team manager history
		const managerYears = Object.keys(
			teamManagers?.teamManagersMap || {}
		);

		for (const year of managerYears) {
			if (Number(year)) {
				years.add(Number(year));
			}
		}

		// Always include the current league season
		const currentSeason = Number(teamManagers?.currentSeason);

		if (currentSeason) {
			years.add(currentSeason);
		}

		return [...years].sort((a, b) => a - b);
	}


	function getPodiumForYear(podiums, year) {
		return (podiums || []).find(
			(podium) => Number(podium?.year) === Number(year)
		);
	}
</script>


<style>

	/* =================================
	   PAGE
	================================= */

	.history-page {
		width: 100%;

		min-height: 100vh;

		padding: 45px 35px 80px;

		position: relative;
	}


	/* =================================
	   MAIN WHITE CLOUD PANEL
	================================= */

	.history-card {
		position: relative;

		width: min(94%, 1100px);

		margin: 30px auto 60px;

		padding: 42px 45px 55px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.97),
				rgba(249, 252, 255, 0.94)
			);

		border: 1px solid rgba(255, 255, 255, 0.95);

		border-radius: 32px;

		box-shadow:
			0 12px 40px rgba(37, 77, 150, 0.16);

		overflow: hidden;

		z-index: 1;
	}


	/* =================================
	   CLOUD EFFECT
	================================= */

	.history-card::before {
		content: "";

		position: absolute;

		width: 360px;
		height: 190px;

		top: -90px;
		left: -80px;

		background: rgba(255, 255, 255, 0.82);

		border-radius: 50%;

		filter: blur(12px);

		pointer-events: none;
	}

	.history-card::after {
		content: "";

		position: absolute;

		width: 400px;
		height: 220px;

		right: -120px;
		bottom: -120px;

		background: rgba(255, 255, 255, 0.80);

		border-radius: 50%;

		filter: blur(13px);

		pointer-events: none;
	}


	/* =================================
	   HEADING
	================================= */

	.history-title {
		position: relative;

		z-index: 5;

		margin: 0 0 25px;

		text-align: center;

		font-family: 'Luckiest Guy', cursive;

		font-size: 44px;

		font-weight: 400;

		line-height: 1.1;

		letter-spacing: 0.5px;

		color: #35116b;
	}


	/* =================================
	   YEAR SELECTOR
	================================= */

	.year-selector {
		position: relative;

		z-index: 10;

		display: flex;

		align-items: center;

		justify-content: center;

		gap: 8px;

		margin: 0 auto 35px;

		flex-wrap: wrap;
	}


	.year-button {
		border: none;

		cursor: pointer;

		font-family: 'Luckiest Guy', cursive;

		font-size: 15px;

		letter-spacing: 0.3px;

		color: #45618f;

		background:
			rgba(239, 246, 255, 0.9);

		border: 1px solid rgba(73, 126, 213, 0.18);

		border-radius: 14px;

		padding: 10px 18px;

		box-shadow:
			0 3px 8px rgba(51, 93, 166, 0.06);

		transition:
			transform 0.18s ease,
			background 0.18s ease,
			color 0.18s ease,
			box-shadow 0.18s ease;
	}

	.year-button:hover {
		transform: translateY(-2px);

		background:
			#eef7ff;

		box-shadow:
			0 5px 12px rgba(51, 93, 166, 0.12);
	}

	.year-button.active {
		background:
			linear-gradient(
				135deg,
				#6d28d9,
				#315fc5
			);

		color: white;

		border-color: transparent;

		box-shadow:
			0 5px 15px rgba(69, 48, 150, 0.22);

		transform: translateY(-1px);
	}


	/* =================================
	   AWARDS CONTENT
	================================= */

	.awards-content {
		position: relative;

		z-index: 5;

		width: 100%;
	}


	/* =================================
	   NO AWARDS
	================================= */

	.no-awards {
		position: relative;

		z-index: 5;

		max-width: 650px;

		margin: 60px auto 30px;

		padding: 35px;

		text-align: center;

		background:
			rgba(255, 255, 255, 0.78);

		border: 1px solid rgba(73, 126, 213, 0.14);

		border-radius: 22px;

		box-shadow:
			0 5px 20px rgba(37, 77, 150, 0.08);

		font-family: 'Poppins', sans-serif;

		font-size: 17px;

		line-height: 1.6;

		color: #40577f;
	}


	/* =================================
	   LOADING
	================================= */

	.loading {
		position: relative;

		z-index: 5;

		width: 85%;

		max-width: 500px;

		margin: 80px auto;

		padding: 25px;

		text-align: center;

		font-family: 'Poppins', sans-serif;

		color: #40577f;
	}


	/* =================================
	   ERROR
	================================= */

	.error-message {
		position: relative;

		z-index: 5;

		width: min(90%, 700px);

		margin: 60px auto;

		padding: 25px;

		background: rgba(255, 255, 255, 0.9);

		border-radius: 20px;

		text-align: center;

		font-family: 'Poppins', sans-serif;

		color: #40577f;
	}


	/* =================================
	   RESPONSIVE
	================================= */

	@media (max-width: 800px) {

		.history-page {
			padding: 30px 15px 60px;
		}

		.history-card {
			width: 96%;

			padding: 32px 22px 45px;

			border-radius: 25px;
		}

		.history-title {
			font-size: 36px;
		}

		.year-button {
			font-size: 14px;

			padding: 9px 14px;
		}
	}


	@media (max-width: 500px) {

		.history-page {
			padding: 20px 8px 40px;
		}

		.history-card {
			padding: 25px 12px 35px;

			border-radius: 20px;
		}

		.history-title {
			font-size: 30px;
		}

		.year-selector {
			gap: 5px;

			margin-bottom: 25px;
		}

		.year-button {
			font-size: 13px;

			padding: 8px 12px;
		}
	}

</style>


<div class="history-page">

	<div class="history-card">

		<div class="cloud"></div>


		<h1 class="history-title">
			LEAGUE HISTORY
		</h1>


		{#await waitForAll(awardsData, teamManagersData)}

			<div class="loading">
				<p>Retrieving league history...</p>

				<LinearProgress indeterminate />
			</div>


		{:then [podiums, leagueTeamManagers]}

			{@const availableYears = getAvailableYears(
				podiums,
				leagueTeamManagers
			)}

			{@const activeYear =
				selectedYear ?? availableYears[availableYears.length - 1]}

			{@const selectedPodium =
				getPodiumForYear(podiums, activeYear)}


			<!-- YEAR TOGGLE -->

			<div class="year-selector">

				{#each availableYears as year}

					<button
						type="button"
						class:active={Number(activeYear) === Number(year)}
						class="year-button"
						onclick={() => selectedYear = year}
					>
						{year}
					</button>

				{/each}

			</div>


			<!-- SELECTED YEAR -->

			<div class="awards-content">

				{#if selectedPodium}

					<Awards
						podium={selectedPodium}
						leagueTeamManagers={leagueTeamManagers}
					/>

				{:else}

					<div class="no-awards">

						<strong>{activeYear}</strong>

						<p>
							This season has not been completed yet,
							so no awards have been earned.
						</p>

					</div>

				{/if}

			</div>


		{:catch error}

			<div class="error-message">
				Something went wrong: {error.message}
			</div>

		{/await}

	</div>

</div>
