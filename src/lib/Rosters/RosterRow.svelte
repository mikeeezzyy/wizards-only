<script>
	import { Row, Cell } from '@smui/data-table';
	
	export let player;

	const playerSLotClass = player.slot.replace('/', '').replace('SUPER_', 'S-').replace('REC_', 'R-');
	const playerSlot = player.slot.replace('SUPER_', 'S ').replace('REC_', 'R ');
</script>

<style>
	.playerAvatar {
		position: relative;
		vertical-align: middle;
		height: 40px;
		width: 40px;
		background-position: center;
		background-repeat: no-repeat;
		background-size: auto 45px;
		overflow: visible;
	}

	.teamLogo {
		width: 20px;
		position: absolute;
		top: -1px;
		right: -13px;
		filter: drop-shadow(0 1px 2px rgba(0,0,0,.18));
	}

	.pos {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		border-radius: 8px;
		width: 46px;
		height: 29px;
		font-size: 0.78rem;
		font-weight: 900;
		letter-spacing: .2px;
	}

	.QB { background-color: var(--QB); }
	.WR { background-color: var(--WR); }
	.RB { background-color: var(--RB); }
	.TE { background-color: var(--TE); }
	.FLEX {
		background: linear-gradient(to right, var(--WR), var(--WR) 33.33%, var(--RB) 33.33%, var(--RB) 66.66%, var(--TE) 66.66%);
	}
	.WRRB {
		background: linear-gradient(to right, var(--WR), var(--WR) 50%, var(--RB) 50%);
	}
	.K { background-color: var(--K); }
	.S-FLEX {
		background: linear-gradient(to right, var(--WR), var(--WR) 25%, var(--RB) 25%, var(--RB) 50%, var(--TE) 50%, var(--TE) 75%, var(--QB));
		background-color: #8f66ff;
	}
	.R-FLEX {
		background: linear-gradient(to right, var(--WR), var(--WR) 50%, var(--TE) 50%);
	}
	.DEF { background-color: var(--DEF); }
	.DL { background-color: var(--DL); }
	.LB { background-color: var(--LB); }
	.DB { background-color: var(--DB); }
	.IDP {
		background: linear-gradient(to right, var(--DL), var(--DL) 33.33%, var(--LB) 33.33%, var(--LB) 66.66%, var(--DB) 66.66%);
	}
	.BN { background-color: var(--BN); }

	:global(.slot) {
		width: 54px !important;
		padding: 5px 6px !important;
		text-align: center !important;
	}

	:global(.avatar) {
		width: 48px !important;
		padding: 0 4px !important;
	}

	:global(.playerCell) {
		height: 48px !important;
		overflow-y: hidden !important;
		border-bottom: 1px solid #e8edf7 !important;
	}

	:global(.nameCell) {
		padding: 0 8px !important;
		font-family: "Poppins", sans-serif;
		font-size: 0.82rem;
		color: #253f70;
	}

	:global(.nameCell .info) {
		line-height: 1.15;
		font-weight: 600;
	}

	span.nickname {
		display: block;
		color: #8b96aa;
		font-style: italic;
		font-size: 0.72rem;
		margin-top: 1px;
	}

	:global(.injury) {
		font-style: italic;
		font-weight: 800;
		font-size: 0.68em;
		margin-left: 0.45em;
		vertical-align: super;
	}

	:global(.Q) { color: #c09111; }
	:global(.OUT), :global(.IR), :global(.PUP) { color: #851313; }

	.additionalInfo {
		display: inline-block;
		font-size: 0.72em;
		color: #7a879d;
		font-weight: 500;
	}

	.text-QB { color: var(--QB); }
	.text-WR { color: var(--WR); }
	.text-RB { color: var(--RB); }
	.text-TE { color: var(--TE); }
	.text-K { color: var(--K); }
	.text-DEF { color: var(--DEF); }
	.text-DL { color: var(--DL); }
	.text-LB { color: var(--LB); }
	.text-DB { color: var(--DB); }

	@media (max-width: 460px) {
		:global(.nameCell) { font-size: 0.76rem; }
	}

	@media (max-width: 355px) {
		:global(.slot) {
			width: 48px !important;
			padding: 4px 2px !important;
		}
		:global(.avatar) { width: 44px !important; }
		.pos {
			width: 42px;
			height: 25px;
			font-size: 0.72em;
		}
		:global(.nameCell) { font-size: 0.7rem; }
	}
</style>

<Row>
	<Cell class="slot playerCell"><span class="pos {playerSLotClass}">{playerSlot}</span></Cell>
	{#if player.avatar}
		<Cell class="avatar playerCell">
            <div class="playerAvatar" style="{player.avatar}">
                {#if player.team && player.poss != "DEF"}
                    <img src="https://sleepercdn.com/images/team_logos/nfl/{player.team.toLowerCase()}.png" class="teamLogo" alt="team logo"/>
                {/if}
            </div>
        </Cell>
	{/if}
	<Cell class="playerCell nameCell" colspan={player.name != "Empty" ? 1 : 3}>
        <div class="info">
            <!-- name -->
            {@html player.name}
            <!-- name -->
            {#if player.poss !== "DEF" && player.team}
                <span class="additionalInfo">
                    &nbsp;-&nbsp;{player.team}
                </span>
            {/if}
        </div>
        {#if player.nickname}
            <span class="nickname">"{player.nickname}</span>
        {/if}
    </Cell>
</Row>