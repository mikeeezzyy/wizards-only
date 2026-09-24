<script>
	import { gotoManager } from '$lib/utils/helper';
  	import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
	import { Icon } from '@smui/icon-button';
	import RosterRow from "./RosterRow.svelte"
	
	export let roster, leagueTeamManagers, startersAndReserve, players, rosterPositions, division, expanded;

	$: team = leagueTeamManagers.teamManagersMap[leagueTeamManagers.currentSeason][roster.roster_id].team;

	let i = 0;

	const digestData = (passedPlayers, rawPlayers, startingPlayers = false, reserve = false) => {
		let digestedRoster = [];
	
		for(const singlePlayer of rawPlayers) {
			if(!startingPlayers && !reserve && startersAndReserve.includes(singlePlayer)) {
				continue;
			}
			let player = {};
			let slot = "BN"
			if(startingPlayers) {
				slot = rosterPositions[i] == "WRRB_FLEX" ? "WR/RB" : rosterPositions[i];
			}

			if(singlePlayer == "0") {
				player = {
					name: "Empty",
					poss: null,
					team: null,
					avatar: null,
					slot: slot
				}
				i++;
				digestedRoster.push(player);
				continue;
			}

			let injury = null;
			switch (passedPlayers[singlePlayer].is) {
				case "Questionable":
					injury = "Q";
					break;
				case "Out":
					injury = "OUT";
					break;
				case "PUP":
					injury = "PUP";
					break;
				case "IR":
					injury = "IR";
					break;
			
				default:
					break;
			}
			player = {
				name: `${passedPlayers[singlePlayer].fn} ${passedPlayers[singlePlayer].ln}${injury ? `<span class="injury ${injury}">${injury}</span>` : ""}`,
                nickname: roster.metadata && roster.metadata[`p_nick_${singlePlayer}`] ? roster.metadata[`p_nick_${singlePlayer}`] : null,
				poss: passedPlayers[singlePlayer].pos,
				team: passedPlayers[singlePlayer].t,
				avatar: passedPlayers[singlePlayer].pos == "DEF" ? `background-image: url(https://sleepercdn.com/images/team_logos/nfl/${singlePlayer.toLowerCase()}.png)` : `background-image: url(https://sleepercdn.com/content/nfl/players/thumb/${singlePlayer}.jpg), url(https://sleepercdn.com/images/v2/icons/player_default.webp)`,
				slot: slot
			}
			i++;
			digestedRoster.push(player);
		}
		i = 0;

		return digestedRoster;
	}

	$: finalStarters = digestData(players, roster.starters, true);
	let finalBench = [];
	$: if(roster.players) {
		finalBench = digestData(players, roster.players);
	}
	let finalIR = null;
	if(roster.reserve) {
		finalIR = digestData(players, roster.reserve, false, true);
	}

	let selected = "0px";
	let status = "minimized";
	const toggleSelected = () => {
		selected = selected == "0px" ? calcHeight() + "px" : "0px";
		status = status == "minimized" ? "expanded" : "minimized";
	}

	let innerWidth;

	const calcHeight = () => {
		const multiplier = 52;
		const benchLength = finalBench.length * multiplier + 53;
		let irLength = 0;
		if(finalIR) {
			irLength = finalIR.length * multiplier + 52;
		}
		return benchLength + irLength;
	}

	$: {
		selected = expanded ? calcHeight() + "px" : "0px";
		status = expanded ? "expanded" : "minimized";
	}

</script>

<svelte:window bind:innerWidth={innerWidth} />

<style>
	h5 {
		text-align: center;
		margin: 0.2em auto;
	}

	.teamAvatar {
		vertical-align: middle;
		border-radius: 50%;
		height: 42px;
		width: 42px;
		object-fit: cover;
		margin-right: 12px;
		border: 2px solid rgba(255,255,255,0.85);
		box-shadow: 0 3px 8px rgba(30,64,175,0.18);
	}

	.team {
		margin: 0;
		width: 100%;
		max-width: 430px;
	}

	:global(.clickable) {
		cursor: pointer;
	}

	:global(.teamInner) {
		display: block;
		margin: 0 auto;
		width: 100% !important;
		border-radius: 18px !important;
		overflow: hidden !important;
		background: #fff !important;
		box-shadow: 0 8px 24px rgba(44,69,145,0.14) !important;
		border: 1px solid rgba(73,126,213,0.16);
	}

	.rosterBench {
		overflow: hidden;
		width: 100%;
		display: block;
		transition: max-height 0.7s ease-in-out;
	}

	:global(.r_1),
	:global(.r_2),
	:global(.r_3) {
		text-align: left;
		background: linear-gradient(135deg, #285fc4, #5b189c) !important;
		color: #fff !important;
	}

	h3 {
		display: flex;
		align-items: center;
		font-size: 1.18rem;
		font-weight: 800;
		line-height: 1.1;
		margin: 0;
		padding: 9px 6px;
		color: #fff;
		font-family: "Poppins", sans-serif;
	}

	h5 {
		font-size: 0.95rem;
		font-weight: 800;
		text-align: center;
		margin: 9px 0;
	}

	:global(.icon) {
		vertical-align: middle;
		margin-right: 5px;
	}

	:global(.interactive) {
		vertical-align: middle;
		cursor: pointer;
	}

	:global(.bench) {
		background-color: #f5f8ff !important;
	}

	.italic {
		color: rgba(255,255,255,0.72);
		font-style: italic;
		font-weight: 600;
	}

	@media (max-width: 900px) {
		.team {
			max-width: 420px;
		}
	}

	@media (max-width: 500px) {
		.team {
			max-width: 100%;
		}

		h3 {
			font-size: 1.05rem;
		}
	}
</style>

<div class="team">
	<DataTable class="teamInner" table$aria-label="Team Name" style="width: {innerWidth * 0.95 > 380 ? 380 : innerWidth * 0.95}px;" >
		<Head> <!-- Team name  -->
			<Row>
				<Cell colspan=4 class="r_{division} clickable">
					<h3 onclick={() => gotoManager({leagueTeamManagers, rosterID: roster.roster_id})}>
						<img alt="team avatar" class="teamAvatar" src="{team ? team.avatar : 'https://sleepercdn.com/images/v2/icons/player_default.webp'}" />
						{team?.name ? team.name : 'No Manager'}
					</h3>
				</Cell>
			</Row>
		</Head>
		<Body>
			<!-- 	Starters	 -->
			{#each finalStarters as starter}
				<RosterRow player={starter} />
			{/each}
			<Row class="interactive" onclick={toggleSelected}>
				<Cell colspan=4 class="{division}"><h5><Icon class="material-icons icon">king_bed</Icon> Bench <span class="italic">({status})</span></h5></Cell>
			</Row>
		</Body>
	</DataTable>
	<div class="rosterBench" style="max-height: {selected}">
		<DataTable class="teamInner" >
			<Body class="bench">
				<!-- 	Bench	 -->
				{#each finalBench as bench}
					<RosterRow player={bench} />
				{/each}
				
				<!-- 	IR	 -->
				{#if finalIR}
					<Row>
					<Cell colspan=4 ><h5><Icon class="material-icons icon">healing</Icon> Injured Reserve</h5></Cell>
					</Row>
					{#each finalIR as ir}
						<RosterRow player={ir} />
					{/each}
				{/if}
				<Row class="interactive" onclick={toggleSelected}>
					<Cell colspan=4 class="{division}"><h5><Icon class="material-icons icon">close_fullscreen</Icon>Close Bench</h5></Cell>
				</Row>
			</Body>
		</DataTable>
	</div>
</div>
