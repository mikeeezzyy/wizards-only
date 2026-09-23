<script>
  	import { getTeamNameFromTeamManagers } from '$lib/utils/helperFunctions/universalFunctions';
    import {Row, Cell } from '@smui/data-table';
    export let draftRow, draftType, row, reversalRound, previous=false, players, year, leagueTeamManagers;
</script>

<style>
    :global(.draftCell) {
        position: relative;
        overflow: hidden;
    }

    :global(.changedHands) {
        background: linear-gradient(
            135deg,
            rgba(239, 232, 255, 0.92),
            rgba(249, 246, 255, 0.92)
        );
    }

    .draftPos {
        position: absolute;
        top: 7px;
        left: 7px;
        z-index: 2;
        font-style: italic;
        font-size: 8px;
        font-weight: 600;
        color: #8b98b3;
    }

    .draftPosPrev {
        position: absolute;
        top: 5px;
        left: 5px;
        z-index: 2;
        font-style: italic;
        font-size: 8px;
        color: #65728e;
    }

    .newOwner {
        position: absolute;
        left: 4px;
        right: 4px;
        bottom: 7px;
        font-family: 'Poppins', sans-serif;
        font-size: 9px;
        font-weight: 600;
        color: #304678;
        text-align: center;
        white-space: break-spaces;
        line-height: 1.15em;
    }

    /* Position color-coding for previous drafts. !important keeps these colors
       visible over SMUI DataTable's default cell background. */
    :global(.draftCell.prevQB) { background: rgba(239, 83, 80, 0.28) !important; }
    :global(.draftCell.prevRB) { background: rgba(76, 175, 80, 0.28) !important; }
    :global(.draftCell.prevWR) { background: rgba(66, 133, 244, 0.28) !important; }
    :global(.draftCell.prevTE) { background: rgba(255, 193, 7, 0.30) !important; }
    :global(.draftCell.prevK) { background: rgba(171, 71, 188, 0.26) !important; }
    :global(.draftCell.prevDEF) { background: rgba(117, 117, 117, 0.22) !important; }
    :global(.draftCell.prevCB) { background: rgba(38, 166, 154, 0.26) !important; }
    :global(.draftCell.prevSS) { background: rgba(63, 81, 181, 0.25) !important; }
    :global(.draftCell.prevFS) { background: rgba(30, 136, 229, 0.25) !important; }
    :global(.draftCell.prevDE) { background: rgba(117, 117, 117, 0.24) !important; }
    :global(.draftCell.prevDL) { background: rgba(117, 117, 117, 0.24) !important; }
    :global(.draftCell.prevLB) { background: rgba(92, 107, 192, 0.25) !important; }

    .playerAvatar {
        display: inline-block;
        position: absolute;
        transform: translate(-50%, -50%);
        left: 50%;
        top: 43%;
        height: 38px;
        width: 38px;
        background-position: center;
        border-radius: 100%;
        background-repeat: no-repeat;
        background-size: auto 38px;
        border: 1px solid rgba(80, 100, 150, 0.12);
    }

    .name {
        display: block;
        width: calc(100% - 8px);
        text-align: center;
        position: absolute;
        left: 4px;
        white-space: break-spaces;
        line-height: 1.05em;
        bottom: 7px;
        color: rgba(38, 59, 100, 0.90);
        font-family: 'Poppins', sans-serif;
        font-size: 9px;
        font-weight: 600;
    }
</style>

<Row>
    {#each draftRow as draftCol, col}
        {#if !previous || draftCol}
            <Cell class="draftCell{draftCol ? ' changedHands' : ''}{previous ? ` prev${players[draftCol.player].pos}` : ''}">
                <span class="draftPos{previous ? "Prev" : ""}">
                    {#if draftType == "auction" && previous}
                        ${draftCol.amount}
                    {:else if draftType == "snake" && !reversalRound}
                        {row}.{row % 2 == 0 ? draftRow.length - col : col + 1}{draftCol?.newOwner ? ` ${getTeamNameFromTeamManagers(leagueTeamManagers, draftCol.newOwner, year)}` : ''}
                    {:else if draftType == "snake" && reversalRound}
                        {#if (row < reversalRound && row % 2 == 0) || (row >= reversalRound && row % 2 == 1)}
                            {row}.{draftRow.length - col}
                        {:else}
                            {row}.{col + 1}
                        {/if}
                        {draftCol?.newOwner ? ` ${getTeamNameFromTeamManagers(leagueTeamManagers, draftCol.newOwner, year)}` : ''}
                    {:else}
                        {#if !reversalRound || row < reversalRound}
                            {row}.{col+1}{draftCol?.newOwner ? ` ${getTeamNameFromTeamManagers(leagueTeamManagers, draftCol.newOwner, year)}` : ''}
                        {:else}
                            {row}.{draftRow.length - col}{draftCol?.newOwner ? ` ${getTeamNameFromTeamManagers(leagueTeamManagers, draftCol.newOwner, year)}` : ''}
                        {/if}
                    {/if}
                </span>
                {#if draftCol && !previous}
                    <div class="newOwner">{getTeamNameFromTeamManagers(leagueTeamManagers, draftCol)}</div>
                {/if}
                {#if previous}
                    <div class="playerAvatar" style="{players[draftCol.player].pos == "DEF" ? `background-image: url(https://sleepercdn.com/images/team_logos/nfl/${draftCol.player.toLowerCase()}.png)` : `background-image: url(https://sleepercdn.com/content/nfl/players/thumb/${draftCol.player}.jpg), url(https://sleepercdn.com/images/v2/icons/player_default.webp)`}" />
                    <br />
                    <div class="name">{`${players[draftCol.player].fn} ${players[draftCol.player].ln}`}{players[draftCol.player].pos == "DEF" ? "" : ` (${players[draftCol.player].t})`}</div>
                {/if}
            </Cell>
        {/if}
    {/each}
</Row>