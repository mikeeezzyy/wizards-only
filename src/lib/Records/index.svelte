<script>
    import Button, { Group, Label } from '@smui/button';
    import { getLeagueRecords, getLeagueTransactions } from '$lib/utils/helper';
    import AllTimeRecords from './AllTimeRecords.svelte';
    import PerSeasonRecords from './PerSeasonRecords.svelte';

    let {leagueData, totals, stale, leagueTeamManagers} = $props();;

    const refreshTransactions = async () => {
        const newTransactions = await getLeagueTransactions(false, true);
        totals = newTransactions.totals;
    }

    let leagueManagerRecords = $state();
    let leagueRosterRecords = $state();
    let leagueWeekHighs = $state();
    let leagueWeekLows = $state();
    let allTimeClosestMatchups = $state();
    let allTimeBiggestBlowouts = $state();
    let mostSeasonLongPoints = $state();
    let leastSeasonLongPoints = $state();
    let seasonWeekRecords = $state();
    let currentYear = $state();
    let lastYear = $state();

    const refreshRecords = async () => {
        const newRecords = await getLeagueRecords(true);

        // update values with new data
        leagueData = newRecords;
    }

    let key = $state("regularSeasonData");

    $effect(() => {
        if(!leagueData || !leagueData[key]) return;

        const selectedLeagueData = leagueData[key];

        leagueManagerRecords = selectedLeagueData.leagueManagerRecords;
        leagueRosterRecords = selectedLeagueData.leagueRosterRecords;
        leagueWeekHighs = selectedLeagueData.leagueWeekHighs;
        leagueWeekLows = selectedLeagueData.leagueWeekLows;
        allTimeClosestMatchups = selectedLeagueData.allTimeClosestMatchups;
        allTimeBiggestBlowouts = selectedLeagueData.allTimeBiggestBlowouts;
        mostSeasonLongPoints = selectedLeagueData.mostSeasonLongPoints;
        leastSeasonLongPoints = selectedLeagueData.leastSeasonLongPoints;
        seasonWeekRecords = selectedLeagueData.seasonWeekRecords;
        currentYear = selectedLeagueData.currentYear;
        lastYear = selectedLeagueData.lastYear;
    });

    if(stale) {
        refreshTransactions();
    }

    if(leagueData.stale) {
        refreshRecords();
    }

    let display = $state("allTime");

</script>

<style>
    .records-page {
        width: 100%;
        min-height: 100vh;
        padding: 34px 14px 42px;
    }

    .records-header {
        width: min(100%, 1445px);
        min-height: 220px;
        margin: 0 auto 18px;
        padding: 34px 24px 28px;
        border-radius: 28px;
        background: linear-gradient(
            135deg,
            rgba(255,255,255,0.94),
            rgba(244,249,255,0.88)
        );
        border: 1px solid rgba(73,126,213,0.18);
        box-shadow: 0 10px 30px rgba(31,58,135,0.16);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
    }

    .records-title {
        margin: 0;
        font-family: 'Luckiest Guy', cursive;
        font-size: clamp(54px, 6vw, 78px);
        font-weight: 400;
        letter-spacing: 1px;
        color: #5b189c;
        line-height: .95;
    }

    .records-subtitle {
        margin: 18px 0 0;
        color: #64748b;
        font-size: clamp(15px, 1.8vw, 20px);
        font-weight: 800;
        letter-spacing: 2.4px;
        text-transform: uppercase;
    }

    .rankingsWrapper {
        width: min(100%, 1445px);
        margin: 0 auto;
    }

    .records-controls {
        margin: 0 auto 24px;
        padding: 28px 24px;
        border-radius: 28px;
        background: rgba(255,255,255,0.94);
        border: 1px solid rgba(73,126,213,0.15);
        box-shadow: 0 9px 26px rgba(31,58,135,0.10);
    }

    .records-control-row {
        display: flex;
        justify-content: center;
        align-items: stretch;
        gap: 0;
        margin: 0 auto 10px;
        max-width: 760px;
    }

    .records-control-row:last-child {
        margin-bottom: 0;
    }

    .control-button {
        flex: 1 1 0;
        min-height: 76px;
        border: 1px solid #d1dbea;
        background: rgba(255,255,255,0.9);
        color: #35598f;
        font-family: 'Roboto', sans-serif;
        font-size: clamp(15px, 1.5vw, 20px);
        font-weight: 900;
        letter-spacing: .2px;
        cursor: pointer;
        transition: .18s ease;
    }

    .control-button:first-child {
        border-radius: 14px 0 0 14px;
    }

    .control-button:last-child {
        border-radius: 0 14px 14px 0;
    }

    .control-button:hover {
        background: #f4f8ff;
    }

    .control-button.active {
        color: #fff;
        border-color: transparent;
        background: linear-gradient(135deg, #6d28d9, #3478ee);
        box-shadow: 0 8px 18px rgba(82,67,214,0.22);
    }

    .empty {
        margin: 10em 0 4em;
        text-align: center;
    }

    @media (max-width: 700px) {
        .records-page {
            padding: 18px 10px 30px;
        }

        .records-header {
            min-height: 175px;
            padding: 26px 16px 22px;
            border-radius: 22px;
        }

        .records-controls {
            padding: 18px 12px;
            border-radius: 22px;
        }

        .records-control-row {
            max-width: none;
        }

        .control-button {
            min-height: 62px;
            font-size: 14px;
            padding: 0 8px;
        }
    }

    @media (max-width: 430px) {
        .records-title {
            font-size: 48px;
        }

        .records-subtitle {
            font-size: 11px;
            letter-spacing: 1.5px;
        }

        .control-button {
            min-height: 54px;
            font-size: 11px;
        }
    }
</style>

<div class="records-page">
    <div class="records-header">
        <h1 class="records-title">RECORDS</h1>
        <p class="records-subtitle">Where the magic lives forever</p>
    </div>

    <div class="rankingsWrapper">
        <div class="records-controls">
            <div class="records-control-row">
                <button
                    class:active={key == "regularSeasonData"}
                    class="control-button"
                    type="button"
                    onclick={() => key = "regularSeasonData"}
                >
                    Regular Season
                </button>
                <button
                    class:active={key == "playoffData"}
                    class="control-button"
                    type="button"
                    onclick={() => key = "playoffData"}
                >
                    Playoffs
                </button>
            </div>

            <div class="records-control-row">
                <button
                    class:active={display == "allTime"}
                    class="control-button"
                    type="button"
                    onclick={() => display = "allTime"}
                >
                    All-Time Records
                </button>
                <button
                    class:active={display == "season"}
                    class="control-button"
                    type="button"
                    onclick={() => display = "season"}
                >
                    Season Records
                </button>
            </div>
        </div>

        {#if display == "allTime"}
            {#if leagueWeekHighs?.length}
                <AllTimeRecords transactionTotals={totals} {allTimeClosestMatchups} {allTimeBiggestBlowouts} {leagueManagerRecords} {leagueWeekHighs} {leagueWeekLows} {leagueTeamManagers} {mostSeasonLongPoints} {leastSeasonLongPoints} {key} />
            {:else}
                <p class="empty">No records <i>yet</i>...</p>
            {/if}
        {:else}
            <PerSeasonRecords transactionTotals={totals} {leagueRosterRecords} {seasonWeekRecords} {leagueTeamManagers} {currentYear} {lastYear} {key} />
        {/if}
    </div>
</div>
