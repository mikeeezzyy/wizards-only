<script>
	import { gotoManager } from '$lib/utils/helper';
	import {
		getAvatarFromTeamManagers,
		getNestedTeamNamesFromTeamManagers
	} from '$lib/utils/helperFunctions/universalFunctions';

	export let podium;
	export let leagueTeamManagers;

	const {
		year,
		champion,
		second,
		third,
		divisions,
		toilet
	} = podium;
</script>


<style>

	/* =================================
	   MAIN AWARDS
	================================= */

	.awards {
		display: block;

		position: relative;

		width: 100%;

		z-index: 5;

		font-family: 'Poppins', sans-serif;

		color: #284a83;
	}


	/* =================================
	   YEAR
	================================= */

	.awards-year {
		margin: 0 0 20px;

		text-align: center;

		font-family: 'Luckiest Guy', cursive;

		font-size: 30px;

		font-weight: 400;

		line-height: 1.1;

		letter-spacing: 0.5px;

		color: #285fc4;
	}


	/* =================================
	   CHAMPION BANNER
	================================= */

	.banner {
		display: block;

		width: 65%;

		max-width: 450px;

		margin: 15px auto 0;
	}


	/* =================================
	   PODIUM
	================================= */

	#podium {
		width: 600px;

		height: 500px;

		position: relative;

		margin: 10px auto 30px;

		max-width: 100%;

	}


	.podiumImage {
		position: absolute;

		bottom: 0;

		left: 0;

		width: 100%;

		height: auto;

		z-index: 3;
	}


	.champ {
		position: absolute;

		width: 20%;

		height: auto;

		transform: translate(-50%, -50%);

		border-radius: 100%;

		border: 2px solid rgba(255, 255, 255, 0.95);

		background-color: white;

		box-shadow:
			0 4px 15px rgba(37, 77, 150, 0.18);
	}


	.laurel {
		position: absolute;

		width: 33%;

		height: auto;

		transform: translate(-50%, -50%);

		bottom: 56.6%;

		left: 50%;

		pointer-events: none;

		z-index: 4;
	}


	.first {
		bottom: 70%;

		left: 50%;

		z-index: 5;
	}


	.second {
		bottom: 43%;

		left: 20%;

		z-index: 5;
	}


	.third {
		bottom: 39%;

		left: 80%;

		z-index: 5;
	}


	/* =================================
	   PODIUM LABELS
	================================= */

	.label {
		white-space: nowrap;

		line-height: 1.1em;

		text-align: center;

		min-height: 34px;

		display: flex;

		flex-direction: column;

		justify-content: center;

		position: absolute;

		transform: translate(-50%, -50%);

		padding: 7px 25px;

		background:
			rgba(255, 255, 255, 0.95);

		border: 1px solid rgba(73, 126, 213, 0.18);

		border-radius: 10px;

		box-shadow:
			0 4px 12px rgba(37, 77, 150, 0.10);

		font-family: 'Poppins', sans-serif;

		font-size: 15px;

		font-weight: 600;

		color: #29416d;
	}


	.firstLabel {
		bottom: 60%;

		left: 50%;
	}


	.secondLabel {
		bottom: 40%;

		left: 20%;
	}


	.thirdLabel {
		bottom: 36%;

		left: 80%;
	}


	/* =================================
	   DIVISIONS
	================================= */

	.divisions {
		display: flex;

		justify-content: center;

		gap: 50px;

		flex-wrap: wrap;

		margin-top: 25px;
	}


	.division {
		text-align: center;

		min-width: 170px;
	}


	.division h6 {
		margin: 5px 0 15px;

		font-family: 'Luckiest Guy', cursive;

		font-size: 18px;

		font-weight: 400;

		color: #35116b;
	}


	.leaderBlock {
		position: relative;

		width: 80px;

		height: 119px;

		margin: 15px auto;
	}


	.divisionLeader {
		position: absolute;

		width: 70px;

		height: 70px;

		transform: translate(-50%, 0%);

		top: 0;

		left: 50%;

		border-radius: 100%;

		border: 2px solid white;

		background-color: white;

		z-index: 3;

		box-shadow:
			0 4px 12px rgba(37, 77, 150, 0.12);
	}


	.medal {
		position: absolute;

		width: 40px;

		height: auto;

		transform: translate(-50%, 0%);

		bottom: 0;

		left: 50%;

		z-index: 2;
	}


	.genLabel {
		white-space: nowrap;

		line-height: 1.1em;

		min-height: 34px;

		display: inline-flex;

		flex-direction: column;

		justify-content: center;

		text-align: center;

		margin: 15px auto 20px;

		padding: 7px 22px;

		background:
			rgba(255, 255, 255, 0.95);

		border: 1px solid rgba(73, 126, 213, 0.18);

		border-radius: 10px;

		box-shadow:
			0 4px 12px rgba(37, 77, 150, 0.10);

		font-family: 'Poppins', sans-serif;

		font-size: 14px;

		font-weight: 600;

		color: #29416d;
	}


	/* =================================
	   TOILET BOWL
	================================= */

	.toiletParent {
		width: 100%;

		text-align: center;

		padding: 35px 0 20px;

		margin-top: 35px;

		border-top: 1px solid rgba(73, 126, 213, 0.12);
	}


	.toilet-banner {
		display: block;

		width: 50%;

		max-width: 350px;

		margin: 20px auto 0;
	}


	.toiletBowl {
		position: relative;

		width: 215px;

		height: 190px;

		margin: 10px auto;
	}


	.toiletWinner {
		position: absolute;

		width: 65px;

		height: 65px;

		transform: translate(-50%, 0%);

		top: 20px;

		left: 55%;

		border-radius: 100%;

		border: 2px solid white;

		z-index: 3;

		box-shadow:
			0 4px 12px rgba(37, 77, 150, 0.12);
	}


	.toilet {
		position: absolute;

		width: 100%;

		height: auto;

		transform: translate(-50%, 0%);

		bottom: 0;

		left: 50%;
	}


	/* =================================
	   CLICKABLE
	================================= */

	.clickable {
		cursor: pointer;

		transition:
			transform 0.18s ease,
			box-shadow 0.18s ease;
	}

	.clickable:hover {
		filter: brightness(1.03);
	}


	/* =================================
	   RESPONSIVE
	================================= */

	@media (max-width: 680px) {

		.label {
			padding: 6px 8px;
		}

		.genLabel {
			padding: 6px 8px;
		}
	}


	@media (max-width: 610px) {

		#podium {
			width: 500px;

			height: 417px;
		}

		.firstLabel {
			bottom: 58%;
		}

		.secondLabel {
			bottom: 35%;
		}

		.thirdLabel {
			bottom: 31%;
		}
	}


	@media (max-width: 510px) {

		#podium {
			width: 400px;

			height: 333px;
		}

		.label {
			font-size: 12px;

			padding: 4px 6px;
		}

		.genLabel {
			font-size: 12px;

			padding: 4px 6px;
		}
	}


	@media (max-width: 410px) {

		#podium {
			width: 300px;

			height: 250px;
		}

		.firstLabel {
			bottom: 53%;
		}

		.secondLabel {
			bottom: 31%;
		}

		.thirdLabel {
			bottom: 27%;
		}

		.divisions {
			gap: 20px;
		}
	}


	@media (max-width: 329px) {

		.label {
			font-size: 10px;
		}

		.genLabel {
			font-size: 10px;
		}
	}

</style>


<div class="awards">

	<h2 class="awards-year">
		{year} AWARDS
	</h2>


	<img
		src="/banner.png"
		class="banner"
		alt="The Champion's Cup"
	/>


	<!-- =========================
	     PODIUM
	========================== -->

	<div id="podium">

		<img
			src="/podium.png"
			class="podiumImage"
			alt="podium"
		/>


		<!-- CHAMPION -->

		<img
			src={getAvatarFromTeamManagers(
				leagueTeamManagers,
				champion,
				year
			)}
			class="first champ clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: champion
				})
			}
			alt="champion"
		/>


		<img
			src="/laurel.png"
			class="laurel"
			alt="laurel"
		/>


		<span
			class="label firstLabel clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: champion
				})
			}
		>
			{@html getNestedTeamNamesFromTeamManagers(
				leagueTeamManagers,
				year,
				champion
			)}
		</span>


		<!-- SECOND -->

		<img
			src={getAvatarFromTeamManagers(
				leagueTeamManagers,
				second,
				year
			)}
			class="second champ clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: second
				})
			}
			alt="second place"
		/>


		<span
			class="label secondLabel clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: second
				})
			}
		>
			{@html getNestedTeamNamesFromTeamManagers(
				leagueTeamManagers,
				year,
				second
			)}
		</span>


		<!-- THIRD -->

		<img
			src={getAvatarFromTeamManagers(
				leagueTeamManagers,
				third,
				year
			)}
			class="third champ clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: third
				})
			}
			alt="third place"
		/>


		<span
			class="label thirdLabel clickable"
			onclick={() =>
				gotoManager({
					year,
					leagueTeamManagers,
					rosterID: third
				})
			}
		>
			{@html getNestedTeamNamesFromTeamManagers(
				leagueTeamManagers,
				year,
				third
			)}
		</span>

	</div>


	<!-- =========================
	     DIVISION CHAMPIONS
	========================== -->

	<div class="divisions">

		{#each divisions || [] as division}

			{#if division.rosterID}

				<div class="division">

					{#if division.name}

						<h6>
							{division.name} Division
						</h6>

					{:else}

						<h6>
							Regular Season Champion
						</h6>

					{/if}


					<div class="leaderBlock">

						<img
							src={getAvatarFromTeamManagers(
								leagueTeamManagers,
								division.rosterID,
								year
							)}
							class="divisionLeader clickable"
							onclick={() =>
								gotoManager({
									year,
									leagueTeamManagers,
									rosterID: division.rosterID
								})
							}
							alt="division champion"
						/>


						<img
							src="/medal.png"
							class="medal"
							alt="champion"
						/>

					</div>


					<span
						class="genLabel clickable"
						onclick={() =>
							gotoManager({
								year,
								leagueTeamManagers,
								rosterID: division.rosterID
							})
						}
					>
						{@html getNestedTeamNamesFromTeamManagers(
							leagueTeamManagers,
							year,
							division.rosterID
						)}
					</span>

				</div>

			{/if}

		{/each}

	</div>


	<!-- =========================
	     TOILET BOWL
	========================== -->

	{#if toilet}

		<div class="toiletParent">

			<img
				src="/toilet-banner.png"
				class="toilet-banner"
				alt="The Toilet Bowl"
			/>


			<div class="toiletBowl">

				<img
					src={getAvatarFromTeamManagers(
						leagueTeamManagers,
						toilet,
						year
					)}
					class="toiletWinner clickable"
					onclick={() =>
						gotoManager({
							year,
							leagueTeamManagers,
							rosterID: toilet
						})
					}
					alt="toilet bowl winner"
				/>


				<img
					src="/toilet-bowl-2.png"
					class="toilet"
					alt="toilet bowl"
				/>

			</div>


			<span
				class="genLabel clickable"
				onclick={() =>
					gotoManager({
						year,
						leagueTeamManagers,
						rosterID: toilet
					})
				}
			>
				{@html getNestedTeamNamesFromTeamManagers(
					leagueTeamManagers,
					year,
					toilet
				)}
			</span>

		</div>

	{/if}

</div>
