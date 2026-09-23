<script>
    import { generateGraph, gotoManager, round } from '$lib/utils/helper';

  	import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
	import RecordTeam from './RecordTeam.svelte';
	import BarChart from '$lib/BarChart.svelte';

    export let key, tradesData, waiversData, weekRecords, weekLows, seasonLongRecords, seasonLongLows, showTies, winPercentages, fptsHistories, lineupIQs, prefix, blowouts, closestMatchups, allTime=false, leagueTeamManagers;

    let graphs = [];
    let curGraph = 0;

    const year = allTime ? null : prefix;

    const setGraphs = (wD) => {
        const lineupIQGraph = {
            stats: lineupIQs,
            x: "Lineup IQ",
            stat: "%",
            header: "Manager Lineup IQ",
            field: "iq",
            short: "Lineup IQ"
        }

        const potentialPointsGraph = {
            stats: lineupIQs,
            x: "Points",
            stat: "",
            header: "Potential Points vs Points",
            field: "potentialPoints",
            secondField: "fpts",
            short: "Potential Points"
        }

        const winsGraph = {
            stats: winPercentages,
            x: "Wins",
            stat: "",
            header: "Team Wins",
            field: "wins",
            secondField: "losses",
            short: "Wins/Losses"
        }

        const winPercentagesGraph = {
            stats: winPercentages,
            x: "Win Percentage",
            stat: "%",
            header: "Team Win Percentages",
            field: "percentage",
            short: "Win Percentage"
        }

        const fptsHistoriesGraph = {
            stats: fptsHistories,
            x: "Fantasy Points",
            stat: "",
            header: "Team Fantasy Points",
            field: "fptsFor",
            short: "Fantasy Points"
        }

        const tradesGraph = {
            stats: tradesData,
            x: "# of trades",
            stat: "",
            header: "Number of Trades Managers Have Made",
            field: "trades",
            short: "Trades"
        }

        const waiversGraph = {
            stats: wD,
            x: "# of Waiver Moves",
            stat: "",
            header: "Waivers Moves Managers Have Made",
            field: "waivers",
            short: "Waivers"
        }
        const gs = [];

        if(lineupIQs[0]?.potentialPoints) {
            gs.push(generateGraph(lineupIQGraph, year));
        }
        gs.push(generateGraph(winsGraph, year, 5));
        gs.push(generateGraph(winPercentagesGraph, year));
        gs.push(generateGraph(fptsHistoriesGraph, year));
        if(lineupIQs[0]?.potentialPoints) {
            gs.push(generateGraph(potentialPointsGraph, year, 10, 0));
        }
        if(key == "regularSeasonData") {
            gs.push(generateGraph(tradesGraph, year));
            gs.push(generateGraph(waiversGraph, year));
        }

        curGraph = 0;
        graphs = gs;
    }

    const setTransactionsAndGraphs = (wD) => {
        if(wD[0].rosterID) {
            for(let i = 1; i <= waiversData.length; i++) {
                if(!tradesData.find(t => t.rosterID == i)) {
                    tradesData.push({
                        rosterID: i,
                        trades: 0,
                    })
                }
            }
        }
        if(wD[0].managerID) {
            for(const userID in leagueTeamManagers.users) {
                if(!tradesData.find(t => t.managerID == userID)) {
                    tradesData.push({
                        managerID: userID,
                        trades: 0,
                    })
                }
            }
        }
        const transactions = [];

        for(const w of wD) {
            let trades = 0;
            if(tradesData[0].managerID) {
                trades = tradesData.find(t => t.managerID == w.managerID)?.trades || 0;
            } else if(tradesData[0].rosterID) {
                trades = tradesData.find(t => t.rosterID == w.rosterID)?.trades || 0;
            }
            const waivers = w.waivers;
            transactions.push({
                rosterID: w.rosterID,
                managerID: w.managerID,
                trades,
                waivers,
            })
        }

        setGraphs(wD)
        return transactions;
    }


    $: transactions =  setTransactionsAndGraphs(waiversData)
    
    let innerWidth;

</script>

<svelte:window bind:innerWidth={innerWidth} />

<style>
    .record-page-heading {
        font-family: 'Luckiest Guy', cursive;
        color: #6d28d9;
    }

    :global(.records-page .recordTable),
    :global(.records-page .rankingTable) {
        background: rgba(255,255,255,0.97);
        border: 1px solid rgba(73,126,213,0.14);
        border-radius: 18px;
        overflow: hidden;
        box-shadow: 0 8px 24px rgba(30,64,175,0.09);
    }

    :global(.records-page .recordTable) {
        margin: 0;
        width: 100%;
        min-width: 0 !important;
        max-width: 100%;
    }

    :global(.records-page .rankingTable) {
        margin: 0;
        min-width: min(100%, 520px);
    }

    :global(.records-page .recordTable .headerPrimary) {
        background: linear-gradient(135deg, #6d28d9, #2563eb);
        color: white;
        font-family: 'Luckiest Guy', cursive;
        font-size: clamp(.78rem, 1.25vw, 1.05rem);
        font-weight: 400;
        letter-spacing: .35px;
        line-height: 1.15;
        white-space: normal;
        overflow-wrap: anywhere;
        padding: 9px 8px;
    }

    :global(.records-page .recordTable .header:not(.headerPrimary)),
    :global(.records-page .rankingTable .header:not(.headerPrimary)) {
        background: #eef7ff;
        color: #334155;
        font-weight: 800;
    }

    :global(.records-page .recordTable td),
    :global(.records-page .rankingTable td),
    :global(.records-page .recordTable th),
    :global(.records-page .rankingTable th) {
        border-bottom-color: rgba(37,99,235,0.10);
    }

    :global(.records-page .recordTable tr:hover td),
    :global(.records-page .rankingTable tr:hover td) {
        background: #f8fbff;
    }

    :global(.records-page .recordTable .mdc-data-table__cell),
    :global(.records-page .recordTable .mdc-data-table__header-cell) {
        white-space: normal;
        overflow-wrap: anywhere;
        word-break: normal;
    }

    :global(.records-page .recordTable .mdc-data-table__header-cell) {
        vertical-align: middle;
    }

    :global(.records-page .fullFlex) {
        gap: 18px;
        align-items: flex-start;
        margin: 18px auto 34px;
        padding: 0 4px;
    }

    :global(.records-page .fullFlex > .recordTable) {
        flex: 0 1 calc(50% - 9px);
        width: calc(50% - 9px);
        box-sizing: border-box;
    }

    :global(.records-page .rankingHolder) {
        padding: 0 4px;
    }

    :global(.records-page h4) {
        font-family: 'Luckiest Guy', cursive;
        font-size: clamp(25px, 3vw, 34px);
        font-weight: 400;
        letter-spacing: .4px;
        color: #6d28d9;
        margin: 26px 0 14px;
    }

    :global(.records-page .subTitle),
    :global(.records-page .italic) {
        color: #64748b;
    }

    :global(.headerPrimary) {
        background-color: var(--headerPrimary);
        text-align: center;
    }

    .italic {
        display: block;
        font-style: italic;
        font-size: 0.9em;
        color: var(--g999);
    }

    :global(.recordTable) {
        box-shadow: 0px 3px 3px -2px var(--boxShadowOne), 0px 3px 4px 0px var(--boxShadowTwo), 0px 1px 8px 0px var(--boxShadowThree);
        margin: 2em;
    }

    :global(.rankingTable) {
        display: table;
        box-shadow: 0px 3px 3px -2px var(--boxShadowOne), 0px 3px 4px 0px var(--boxShadowTwo), 0px 1px 8px 0px var(--boxShadowThree);
        margin: 2em auto 0.5em;
    }

    .fullFlex {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-around;
        margin: 3em auto 5em;
    }

    .subTitle {
        font-style: italic;
        font-size: 0.7em;
        color: #888;
        line-height: 1.2em;
    }

    .sectionHeading {
        width: min(100%, 760px);
        margin: 28px auto 18px;
        padding: 14px 24px 13px;
        border-radius: 18px;
        background: rgba(255,255,255,0.94);
        border: 1px solid rgba(73,126,213,0.16);
        box-shadow: 0 8px 22px rgba(31,58,135,0.10);
        text-align: center;
        box-sizing: border-box;
    }

    h4 {
        text-align: center;
        margin: 0;
    }

    .sectionSubheading {
        margin-top: 5px;
        color: #475569;
        font-size: 0.72rem;
        font-weight: 900;
        letter-spacing: 1.2px;
        line-height: 1.25;
    }

    :global(.cellName) {
        cursor: pointer;
        line-height: 1.2em;
        padding-left: 8px;
    }

    :global(.differentialName) {
        padding: 0.7em 0;
    }

    :global(.rank) {
        padding-right: 0;
    }

    .vs {
        padding-left: 0.6em;
        margin: 0.5em 0;
    }

    :global(.mdc-data-table__cell, .mdc-data-table__header-cell) {
        border-bottom-color: var(--borderOverride);
    }

    /* Start record table resizing */

    @media (max-width: 900px) {
        :global(.records-page .fullFlex > .recordTable) {
            flex-basis: 100%;
            width: 100%;
        }
    }

    @media (max-width: 510px) {
        :global(.recordTable th) {
            font-size: 0.8em;
            padding: 1px 12px;
        }
        :global(.recordTable td) {
            font-size: 0.8em;
            padding: 1px 12px;
        }

        .vsRecord {
            margin: .6em 0;
        }
    }

    @media (max-width: 480px) {
        :global(.rank) {
            padding: 1px 0 1px 5px !important;
        }
        :global(.rank) {
            padding: 1px 0 1px 5px !important;
        }
    }

    @media (max-width: 460px) {
        :global(.recordTable th) {
            font-size: 0.6em;
            padding: 1px 12px;
        }
        :global(.recordTable td) {
            font-size: 0.6em;
            padding: 1px 12px;
        }
    }

    @media (max-width: 365px) {
        :global(.recordTable th) {
            font-size: 0.5em;
            padding: 1px 8px;
        }
        :global(.recordTable td) {
            font-size: 0.5em;
            padding: 1px 8px;
        }
    }

    @media (max-width: 265px) {
        :global(.recordTable th) {
            font-size: 0.4em;
            padding: 1px 5px;
        }
        :global(.recordTable td) {
            font-size: 0.4em;
            padding: 1px 5px;
        }
    }

    /* END record table resizing */

    /* Start ranking table resizing */

    @media (max-width: 570px) {
        :global(.rankingTable th) {
            font-size: 0.8em;
            max-width: 110px;
            white-space: break-spaces;
            padding: 1px 12px;
        }
        :global(.rankingTable td) {
            font-size: 0.8em;
            max-width: 110px;
            white-space: break-spaces;
            padding: 1px 12px;
        }
    }

    @media (max-width: 410px) {
        :global(.rankingTable th) {
            font-size: 0.6em;
            max-width: 90px;
            white-space: break-spaces;
            padding: 1px 12px;
        }
        :global(.rankingTable td) {
            font-size: 0.6em;
            max-width: 90px;
            white-space: break-spaces;
            padding: 1px 12px;
        }
    }

    @media (max-width: 340px) {
        :global(.rankingTable th) {
            font-size: 0.55em;
            max-width: 80px;
            white-space: break-spaces;
            padding: 1px 6px;
        }
        :global(.rankingTable td) {
            font-size: 0.55em;
            max-width: 80px;
            white-space: break-spaces;
            padding: 1px 6px;
        }
    }

    /* END ranking table resizing */
</style>

<div class="sectionHeading">
    <h4>{prefix} Records</h4>
    <div class="sectionSubheading">
        {#if allTime}
            SINGLE WEEK, SEASON, AND PLAYOFF RECORDS
        {:else if key == "playoffData"}
            PLAYOFF RECORDS
        {:else}
            SEASON RECORDS
        {/if}
    </div>
</div>

<div class="fullFlex">
    {#if weekRecords && weekRecords.length}
        <DataTable class="recordTable">
            <Head>
                <Row class="rTableHeader">
                    <Cell class="header headerPrimary" colspan=4>{prefix} {key == "playoffData" ? "Playoff " : ""}Single Week Scoring Records</Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Manager</Cell>
                    <Cell class="header">Week</Cell>
                    <Cell class="header">Total Points</Cell>
                </Row>
            </Head>
            <Body>
                {#each weekRecords as leagueWeekRecord, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName" onclick={() => gotoManager({year: leagueWeekRecord.year || prefix, leagueTeamManagers, rosterID: leagueWeekRecord.rosterID})}>
                            <RecordTeam {leagueTeamManagers} rosterID={leagueWeekRecord.rosterID} year={allTime ? leagueWeekRecord.year : prefix} />
                        </Cell>
                        <Cell>{allTime ? leagueWeekRecord.year + " " : "" }{key == "regularSeasonData" ? "Week " : ""}{leagueWeekRecord.week}</Cell>
                        <Cell>{round(leagueWeekRecord.fpts)}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}

    {#if weekLows && weekLows.length}
        <DataTable class="recordTable">
            <Head>
                <Row>
                    <Cell class="header headerPrimary" colspan=4>{prefix} {key == "playoffData" ? "Playoff " : ""}Single Week Scoring Lows</Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Manager</Cell>
                    <Cell class="header">Week</Cell>
                    <Cell class="header">Total Points</Cell>
                </Row>
            </Head>
            <Body>
                {#each weekLows as leagueWeekLow, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName" onclick={() => gotoManager({year: leagueWeekLow.year || prefix, leagueTeamManagers, rosterID: leagueWeekLow.rosterID})}>
                            <RecordTeam {leagueTeamManagers} rosterID={leagueWeekLow.rosterID} year={allTime ? leagueWeekLow.year : prefix} />
                        </Cell>
                        <Cell>{allTime ? leagueWeekLow.year + " " : "" }{key == "regularSeasonData" ? "Week " : ""}{leagueWeekLow.week}</Cell>
                        <Cell>{round(leagueWeekLow.fpts)}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}

    {#if allTime && key == "regularSeasonData"}
        <DataTable class="recordTable">
            <Head>
                <Row>
                    <Cell class="header headerPrimary" colspan=5>All-Time Highest Season Points<span class="italic">Ranked by PPG</span></Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Manager</Cell>
                    <Cell class="header">Year</Cell>
                    <Cell class="header">Total Points</Cell>
                    <Cell class="header">PPG</Cell>
                </Row>
            </Head>
            <Body>
                {#each seasonLongRecords as mostSeasonLongPoint, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName" onclick={() => gotoManager({year: mostSeasonLongPoint.year, leagueTeamManagers, rosterID: mostSeasonLongPoint.rosterID})}>
                            <RecordTeam {leagueTeamManagers} rosterID={mostSeasonLongPoint.rosterID} year={mostSeasonLongPoint.year} />
                        </Cell>
                        <Cell>{mostSeasonLongPoint.year}</Cell>
                        <Cell>{round(mostSeasonLongPoint.fpts)}</Cell>
                        <Cell>{mostSeasonLongPoint.fptsPerGame}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}
    
    {#if allTime && key == "regularSeasonData"}
        <DataTable class="recordTable">
            <Head>
                <Row>
                    <Cell class="header headerPrimary" colspan=5>All-Time Lowest Season Points<span class="italic">Ranked by PPG</span></Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Manager</Cell>
                    <Cell class="header">Year</Cell>
                    <Cell class="header">Total Points</Cell>
                    <Cell class="header">PPG</Cell>
                </Row>
            </Head>
            <Body>
                {#each seasonLongLows as leastSeasonLongPoint, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName" onclick={() => gotoManager({year: leastSeasonLongPoint.year, leagueTeamManagers, rosterID: leastSeasonLongPoint.rosterID})}>
                            <RecordTeam {leagueTeamManagers} rosterID={leastSeasonLongPoint.rosterID} year={leastSeasonLongPoint.year} />
                        </Cell>
                        <Cell>{leastSeasonLongPoint.year}</Cell>
                        <Cell>{round(leastSeasonLongPoint.fpts)}</Cell>
                        <Cell>{leastSeasonLongPoint.fptsPerGame}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}

    {#if blowouts && blowouts.length}
        <DataTable class="recordTable">
            <Head>
                <Row>
                    <Cell class="header headerPrimary" colspan=4>{prefix} Largest {key == "playoffData" ? "Playoff " : ""}Blowouts</Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Matchup</Cell>
                    <Cell class="header">Week</Cell>
                    <Cell class="header">Differential</Cell>
                </Row>
            </Head>
            <Body>
                {#each blowouts as blowout, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName differentialName">
                            <div class="vsRecord">
                                <div onclick={() => gotoManager({year: blowout.year || prefix, leagueTeamManagers, rosterID: blowout.home.rosterID})}>
                                    <RecordTeam {leagueTeamManagers} rosterID={blowout.home.rosterID} year={allTime ? blowout.year : prefix} compressed={true} points={round(blowout.home.fpts)} />
                                </div>
                                <p class="vs">
                                    vs
                                </p>
                                <div onclick={() => gotoManager({year: blowout.year || prefix, leagueTeamManagers, rosterID: blowout.away.rosterID})}>
                                    <RecordTeam {leagueTeamManagers} rosterID={blowout.away.rosterID} year={allTime ? blowout.year : prefix} compressed={true} points={round(blowout.away.fpts)} />
                                </div>
                            </div>
                        </Cell>
                        <Cell>{allTime ? blowout.year + " " : "" }{key == "regularSeasonData" ? "Week " : ""}{blowout.week}</Cell>
                        <Cell>{round(blowout.differential)}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}

    {#if closestMatchups && closestMatchups.length}
        <DataTable class="recordTable">
            <Head>
                <Row>
                    <Cell class="header headerPrimary" colspan=4>{prefix} Narrowest {key == "playoffData" ? "Playoff " : ""}Wins</Cell>
                </Row>
                <Row>
                    <Cell class="header rank"></Cell>
                    <Cell class="header">Matchup</Cell>
                    <Cell class="header">Week</Cell>
                    <Cell class="header">Differential</Cell>
                </Row>
            </Head>
            <Body>
                {#each closestMatchups as closestMatchup, ix}
                    <Row>
                        <Cell class="rank">{ix + 1}</Cell>
                        <Cell class="cellName differentialName">
                            <div class="vsRecord">
                                <div onclick={() => gotoManager({year: closestMatchup.year || prefix, leagueTeamManagers, rosterID: closestMatchup.home.rosterID})}>
                                    <RecordTeam {leagueTeamManagers} rosterID={closestMatchup.home.rosterID} year={allTime ? closestMatchup.year : prefix} compressed={true} points={round(closestMatchup.home.fpts)} />
                                </div>
                                <p class="vs">
                                    vs
                                </p>
                                <div onclick={() => gotoManager({year: closestMatchup.year || prefix, leagueTeamManagers, rosterID: closestMatchup.away.rosterID})}>
                                    <RecordTeam {leagueTeamManagers} rosterID={closestMatchup.away.rosterID} year={allTime ? closestMatchup.year : prefix} compressed={true} points={round(closestMatchup.away.fpts)} />
                                </div>
                            </div>
                        </Cell>
                        <Cell>{allTime ? closestMatchup.year + " " : "" }{key == "regularSeasonData" ? "Week " : ""}{closestMatchup.week}</Cell>
                        <Cell>{round(closestMatchup.differential)}</Cell>
                    </Row>
                {/each}
            </Body>
        </DataTable>
    {/if}
</div>

{#if graphs.length}
    <BarChart {graphs} bind:curGraph={curGraph} {leagueTeamManagers} />
{/if}

