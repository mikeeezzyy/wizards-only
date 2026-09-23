<script>
  	import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
	import LinearProgress from '@smui/linear-progress';
    import { onMount } from 'svelte';
    import DraftRow from './DraftRow.svelte';
    import { gotoManager } from '$lib/utils/helper'
	import { getAvatarFromTeamManagers, getTeamNameFromTeamManagers } from '$lib/utils/helperFunctions/universalFunctions';
    
    export let draftData, leagueTeamManagers, previous = false, year, players;

    const {draftOrder, draft, accuracy, reversalRound, draftType} = draftData;

    let progress = 0;
    let closed = false;

    onMount(loadAccuracy);

    function loadAccuracy() {
        if(!accuracy || accuracy === 1) return;
        let timer;
        progress = 0;
        closed = false;
        clearInterval(timer);
        timer = setInterval(() => {
            progress += 0.02;
            if (progress >= accuracy) {
                clearInterval(timer);
                if (progress >= 1) {
                    progress = 1;
                    closed = true;
                }
            }

        }, 100);
    }
</script>

<style>
    .accuracy {
        width: min(92%, 850px);
        margin: 0 auto 22px;
        padding: 14px 18px;
        border: 1px solid rgba(85, 122, 194, 0.16);
        border-radius: 14px;
        background: rgba(241, 248, 255, 0.82);
        box-shadow: 0 3px 12px rgba(50, 85, 150, 0.05);
    }

    .accuracyText {
        margin-bottom: 8px;
        font-family: 'Poppins', sans-serif;
        font-size: 11px;
        font-weight: 700;
        color: #49639c;
    }

    .disclaimer {
        font-style: italic;
        font-weight: 500;
        color: #7c8baa;
    }

    :global(.draftBoard) {
        display: block;
        width: calc(100% - 32px);
        margin: 0 16px 28px;
        overflow-x: auto;
        border: 1px solid rgba(85, 122, 194, 0.18);
        border-radius: 16px;
        background: rgba(255, 255, 255, 0.78);
        box-shadow: 0 4px 15px rgba(50, 85, 150, 0.06);
    }

    :global(.draftBoard table) {
        border-collapse: separate;
        border-spacing: 0;
        table-layout: fixed;
        width: 100%;
        min-width: 1050px;
        overflow: hidden;
    }

    :global(.draftTeam) {
        height: 92px;
        padding: 9px 5px;
        text-align: center;
        vertical-align: middle;
        background: linear-gradient(135deg, #f0f6ff, #faf7ff);
        border-right: 1px solid rgba(85, 122, 194, 0.13);
        border-bottom: 1px solid rgba(85, 122, 194, 0.13);
        color: #304678;
        font-family: 'Poppins', sans-serif;
        font-size: 10px;
        font-weight: 700;
        line-height: 1.15;
        white-space: normal;
    }

    :global(.draftBoard th:last-child .draftTeam),
    :global(.draftBoard .draftTeam:last-child) {
        border-right: none;
    }

    :global(.draftBoard td) {
        position: relative;
        height: 92px;
        padding: 0;
        font-family: 'Poppins', sans-serif;
        font-size: 10px;
        border-right: 1px solid rgba(85, 122, 194, 0.11);
        border-bottom: 1px solid rgba(85, 122, 194, 0.10);
        background: rgba(255, 255, 255, 0.70);
    }

    :global(.draftBoard tr:last-child td) {
        border-bottom: none;
    }

    :global(.draftBoard td:last-child) {
        border-right: none;
    }

    .avatar {
        display: inline-block;
        width: 42px;
        height: 42px;
        margin: 0 0 5px;
        border-radius: 50%;
        object-fit: cover;
        border: 2px solid rgba(122, 80, 220, 0.20);
        box-shadow: 0 2px 6px rgba(50, 85, 150, 0.10);
        vertical-align: middle;
    }

    .clickable {
        cursor: pointer;
    }

    :global(.curDraftName) {
        color: #7c8baa;
        font-size: 8px;
        font-style: italic;
        font-weight: 500;
    }

    @media (max-width: 700px) {
        :global(.draftBoard) {
            width: calc(100% - 18px);
            margin: 0 9px 22px;
        }

        :global(.draftBoard table) {
            min-width: 900px;
        }

        :global(.draftTeam) {
            height: 78px;
            font-size: 9px;
        }

        :global(.draftBoard td) {
            height: 78px;
        }

        .avatar {
            width: 36px;
            height: 36px;
        }
    }
</style>

{#if accuracy && accuracy !== 1 && !closed}
    <div class="accuracy">
        <div class="accuracyText">
            Upcomig draft order accuracy: {parseInt(progress*100)}%
            <span class="disclaimer">(accuracy will improve as the regular season progresses)</span>
        </div>
        <LinearProgress {progress} {closed} />
    </div>
{/if}

<DataTable class="draftBoard">
    <Head>
        <Row>
            {#each draftOrder as draftPosition}
                {#if draftPosition}
                    <Cell class="draftTeam">
                        <img class="avatar clickable" onclick={() => gotoManager({year, leagueTeamManagers, rosterID: draftPosition})} src="{getAvatarFromTeamManagers(leagueTeamManagers, draftPosition, year)}" alt="{getTeamNameFromTeamManagers(leagueTeamManagers, draftPosition, year)} avatar"/>
                        <br />
                        <span class="clickable" onclick={() => gotoManager({year, leagueTeamManagers, rosterID: draftPosition})}>{getTeamNameFromTeamManagers(leagueTeamManagers, draftPosition, year)}{@html getTeamNameFromTeamManagers(leagueTeamManagers, draftPosition, year) != getTeamNameFromTeamManagers(leagueTeamManagers, draftPosition) ? `<br /><span class="curDraftName">(${getTeamNameFromTeamManagers(leagueTeamManagers, draftPosition)})</span>` : ''}</span>
                    </Cell>
                {/if}
            {/each}
        </Row>
    </Head>
    <Body>
        {#each draft as draftRow, row}
            <DraftRow {draftRow} row={row + 1} {previous} {reversalRound} {draftType} {players} {leagueTeamManagers} {year} />
        {/each}
    </Body>
</DataTable>

