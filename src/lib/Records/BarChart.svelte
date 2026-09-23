<script>
    import Button, { Group, Label } from '@smui/button';
	import Bar from './Bar.svelte';

    export let graphs, leagueTeamManagers, curGraph = 0;

    const colors = [
        "--barChartOne",
        "--barChartTwo",
        "--barChartThree",
        "--barChartFour",
        "--barChartFive",
        "--barChartSix",
    ];

    // note that due to changig to horizontal, yMin and yMax are now used as xMin and xMax
    $: xMin = graphs[curGraph].header === "Team Wins" ? graphs[curGraph].xMin : (graphs[curGraph].secondStats.length > 0 ? graphs[curGraph].xMin/3 : graphs[curGraph].xMin);
    $: xMax = graphs[curGraph].xMax;
    $: stats = graphs[curGraph].stats;
    $: secondStats = graphs[curGraph].secondStats;
    $: managerIDs = graphs[curGraph].managerIDs;
    $: rosterIDs = graphs[curGraph].rosterIDs;
    $: labels = graphs[curGraph].labels;
    $: header = graphs[curGraph].header;
    $: year = graphs[curGraph].year;
</script>

<style>
    .chartWrapper {
		background-color: var(--fff);
        padding: 1em 0 0.5em;
        margin: 0 auto;
        max-width: 950px;
        box-shadow: 0px 3px 3px -2px var(--boxShadowOne), 0px 3px 4px 0px var(--boxShadowTwo), 0px 1px 8px 0px var(--boxShadowThree);
    }

    .barChart {
        display: block;
        position: relative;
        width: 100%;
        height: 100%;
    }

    .chart-heading {
        width: min(100%, 1450px);
        min-height: 150px;
        margin: 0 auto 18px;
        padding: 24px 24px 20px;
        border-radius: 28px;
        background: rgba(255,255,255,0.90);
        border: 1px solid rgba(73,126,213,0.18);
        box-shadow: 0 10px 30px rgba(31,58,135,0.14);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        box-sizing: border-box;
    }

    .chart-heading h6 {
        margin: 0;
        font-family: 'Luckiest Guy', cursive;
        font-size: clamp(42px, 5vw, 64px);
        font-weight: 400;
        letter-spacing: 1px;
        color: #5b189c;
        line-height: 1;
        text-transform: uppercase;
    }

    .chart-heading p {
        margin: 14px 0 0;
        color: #475569;
        font-family: inherit;
        font-size: clamp(15px, 1.7vw, 20px);
        font-weight: 800;
        letter-spacing: 2px;
        text-transform: uppercase;
        line-height: 1.2;
    }

    @media (max-width: 700px) {
        .chart-heading {
            min-height: 120px;
            padding: 20px 14px 18px;
            border-radius: 22px;
        }

        .chart-heading h6 {
            font-size: 38px;
        }

        .chart-heading p {
            font-size: 12px;
            letter-spacing: 1.5px;
        }
    }

    .chartWrapper {
        max-width: 95%;
    }
    @media (max-width: 850px) {
        .chartWrapper {
            max-width: 100%;
        }
    }

    .buttonHolderG {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        margin: 0 auto 14px;
        text-align: center;
    }

    .buttonHolderG :global(.mdc-button-group) {
        margin: 0 auto;
    }

    @media (max-width: 525px) {
        :global(.buttonHolderG .selectionButtons) {
            font-size: 0.6em;
            height: 32px;
        }
    }

    @media (max-width: 505px) {
        :global(.buttonHolderG .selectionButtons) {
            height: 26px;
        }
    }

    @media (max-width: 405px) {
        :global(.buttonHolderG .selectionButtons) {
            font-size: 0.5em;
            padding: 0 6px;
        }
    }

    @media (max-width: 260px) {
        :global(.buttonHolderG .selectionButtons) {
            font-size: 0.4em;
            padding: 0 2px;
            height: 24px;
            min-width: 40px;
        }
    }

    /* End button resizing */
</style>

<div class="chart-heading">
    <h6>{year ? `${year} RANKINGS` : "ALL-TIME RANKINGS"}</h6>
    <p>{header}</p>
</div>
{#if graphs.length > 1}
    <div class="buttonHolderG">
        <Group variant="outlined">
            {#each graphs as graph, ix}
                {#if ix < 4}
                    <Button class="selectionButtons" onclick={() => curGraph = ix} variant="{curGraph == ix ? "raised" : "outlined"}">
                        <Label>{graph.short}</Label>
                    </Button>
                {/if}
            {/each}
        </Group>
        <br />
        <Group variant="outlined">
            {#each graphs as graph, ix}
                {#if ix > 3}
                    <Button class="selectionButtons" onclick={() => curGraph = ix} variant="{curGraph == ix ? "raised" : "outlined"}">
                        <Label>{graph.short}</Label>
                    </Button>
                {/if}
            {/each}
        </Group>
    </div>
{/if}

<div class="chartWrapper">
    <div class="barChart" >
        {#each managerIDs as managerID, ix}
            <Bar {leagueTeamManagers} {managerID} rosterID={rosterIDs[ix]} {xMin} {xMax} stat={stats[ix]} secondStat={secondStats[ix]} {year} label={labels.stat} color={colors[ix % colors.length]} recordMode={header === "Team Wins"} />
        {/each}
    </div>
</div>

