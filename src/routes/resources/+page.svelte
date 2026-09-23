<script>
	import { Resources } from '$lib/components';
</script>


<div class="resources-page">

	<section class="resources-card">

		<div class="cloud cloud-one"></div>
		<div class="cloud cloud-two"></div>

		<div class="resources-content">
			<Resources />
		</div>

	</section>

</div>


<style>

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
