<script>
	const resources = [
		{
			title: 'Sleeper',
			href: 'https://sleeper.com'
		},
		{
			title: 'FantasyPros Dynasty Rankings',
			href: 'https://www.fantasypros.com/nfl/rankings/dynasty-overall.php'
		},
		{
			title: 'FantasyCalc Trade Calculator',
			href: 'https://fantasycalc.com/trade-calculator'
		},
		{
			title: 'KeepTradeCut Dynasty Rankings',
			href: 'https://keeptradecut.com/dynasty-rankings?page=0&filters=RB'
		},
		{
			title: 'Dynasty League Football Trade Analyzer',
			href: 'https://dynastyleaguefootball.com/trade-analyzer/'
		},
		{
			title: 'CLICK HERE FOR $500',
			href: 'https://www.youtube.com/watch?v=QlJXhWQZWnQ&t=1s'
		}
	];
</script>

<div class="resources-page">

	<section class="resources-card">

		<div class="cloud cloud-one"></div>
		<div class="cloud cloud-two"></div>

		<div class="resources-content">

			<h1 class="resources-title">
				HELPFUL DYNASTY<br />
				RESOURCES
			</h1>

			<div class="resources-list">

				{#each resources as resource}

					<a
						class="resource-link"
						href={resource.href}
						target="_blank"
						rel="noopener noreferrer"
					>

						<span class="resource-icon" aria-hidden="true">
							{#if resource.title === 'Sleeper'}🏈
							{:else if resource.title === 'FantasyPros Dynasty Rankings'}📊
							{:else if resource.title === 'FantasyCalc Trade Calculator'}🧮
							{:else if resource.title === 'KeepTradeCut Dynasty Rankings'}🎯
							{:else if resource.title === 'Dynasty League Football Trade Analyzer'}🔄
							{:else}💰
							{/if}
						</span>

						<span>{resource.title}</span>

					</a>

				{/each}

			</div>

		</div>

	</section>

</div>

<style>

/* RESOURCES HEADER */
.resources-title {
    margin: 0 auto 38px;
    text-align: center;
    font-family: 'Luckiest Guy', cursive;
    font-size: 42px;
    line-height: 1.05;
    letter-spacing: 1px;
    color: #35116b;
}

/* SIX RESOURCE LINKS */
.resources-list {
    width: 90%;
    max-width: 800px;
    margin: 0 auto;
    border: 1px solid rgba(44, 69, 145, 0.18);
    box-shadow: 0 4px 14px rgba(44, 69, 145, 0.14);
    background: rgba(255, 255, 255, 0.72);
}

.resource-link {
    display: flex;
    align-items: center;
    gap: 18px;
    min-height: 60px;
    padding: 0 24px;
    border-bottom: 1px solid rgba(44, 69, 145, 0.18);
    color: #2c4778;
    font-family: 'Poppins', sans-serif;
    font-size: 18px;
    font-weight: 500;
    text-decoration: none;
    box-sizing: border-box;
}

.resource-link:last-child {
    border-bottom: none;
}

.resource-link:hover {
    background: rgba(235, 241, 255, 0.85);
    color: #5126b8;
}

.resource-icon {
    flex: 0 0 40px;
    width: 40px;
    height: 40px;
    border-radius: 12px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #285fc4, #6d28d9);
    color: #fff;
    font-size: 21px;
    line-height: 1;
    box-shadow: 0 4px 10px rgba(53, 17, 107, 0.20);
    border: 2px solid rgba(255, 255, 255, 0.75);
}

@media (max-width: 700px) {
    .resources-title {
        font-size: 32px;
        margin-bottom: 28px;
    }

    .resources-list {
        width: 100%;
    }

    .resource-link {
        min-height: 54px;
        padding: 0 14px;
        gap: 12px;
        font-size: 15px;
    }
}



	/* =================================
	   PAGE
	================================= */

	.resources-page {
		width: 100%;
		min-height: 100vh;

		padding: 45px 35px 80px;

		position: relative;
	}


	/* =================================
	   WHITE CLOUD PANEL
	================================= */

	.resources-card {
		position: relative;

		width: min(92%, 1000px);

		margin: 35px auto 55px;

		padding: 42px 50px;

		background:
			linear-gradient(
				135deg,
				rgba(255, 255, 255, 0.97),
				rgba(250, 253, 255, 0.94)
			);

		border: 1px solid rgba(80, 130, 220, 0.18);

		border-radius: 30px;

		box-shadow:
			0 12px 40px rgba(37, 77, 150, 0.16);

		overflow: hidden;

		z-index: 1;
	}


	/* =================================
	   CLOUD EFFECT
	================================= */

	.resources-card::before {
		content: "";

		position: absolute;

		width: 260px;
		height: 150px;

		top: -70px;
		left: -60px;

		background: rgba(255, 255, 255, 0.78);

		border-radius: 50%;

		filter: blur(8px);

		pointer-events: none;
	}

	.resources-card::after {
		content: "";

		position: absolute;

		width: 300px;
		height: 170px;

		right: -80px;
		bottom: -80px;

		background: rgba(255, 255, 255, 0.75);

		border-radius: 50%;

		filter: blur(10px);

		pointer-events: none;
	}

	.cloud {
		position: absolute;

		background: rgba(255, 255, 255, 0.65);

		border-radius: 50%;

		filter: blur(7px);

		pointer-events: none;
	}

	.cloud-one {
		width: 180px;
		height: 100px;

		top: 15px;
		right: 100px;
	}

	.cloud-two {
		width: 220px;
		height: 120px;

		bottom: 20px;
		left: 80px;
	}


	/* =================================
	   CONTENT
	================================= */

	.resources-content {
		position: relative;

		z-index: 5;

		width: 100%;

		color: #284a83;

		font-family: 'Poppins', sans-serif;
	}


	/* =================================
	   MATCH LEAGUE INFO TYPOGRAPHY
	================================= */

	.resources-content :global(.banner) {
		margin-bottom: 30px;
	}

	.resources-content :global(.banner h4) {
		margin: 0;

		font-family: 'Luckiest Guy', cursive !important;

		font-size: 42px !important;

		font-weight: 400 !important;

		line-height: 1.1;

		letter-spacing: 0.5px;

		color: #35116b !important;

		text-align: center;

		text-transform: uppercase;
	}


	/* Resource list text */

	.resources-content :global(.linkText) {
		font-family: 'Poppins', sans-serif !important;

		font-size: 18px !important;

		font-weight: 500;

		color: #29416d !important;
	}


	/* Resource links */

	.resources-content :global(a) {
		font-family: 'Poppins', sans-serif !important;

		color: #29416d !important;

		text-decoration: none;
	}


	/* Icons */

	.resources-content :global(.material-icons) {
		color: #29416d !important;
	}


	/* List */

	.resources-content :global(.list) {
		width: 90%;

		max-width: 800px;

		margin: 15px auto;

		border-radius: 0;

		background: rgba(255, 255, 255, 0.45);
	}


	/* =================================
	   RESPONSIVE
	================================= */

	@media (max-width: 800px) {

		.resources-page {
			padding: 30px 15px 60px;
		}

		.resources-card {
			width: 96%;

			padding: 30px 25px;

			border-radius: 24px;
		}

		.resources-content :global(.banner h4) {
			font-size: 34px !important;
		}

		.resources-content :global(.linkText) {
			font-size: 16px !important;
		}
	}


	@media (max-width: 500px) {

		.resources-page {
			padding: 20px 8px 40px;
		}

		.resources-card {
			padding: 25px 18px;

			border-radius: 20px;
		}

		.resources-content :global(.banner h4) {
			font-size: 28px !important;
		}

		.resources-content :global(.linkText) {
			font-size: 15px !important;
		}
	}


</style>
