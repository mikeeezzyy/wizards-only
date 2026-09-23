<script>
    import { onMount } from 'svelte';
    import { getLeagueRecords, getLeagueTransactions, getLeagueTeamManagers } from '$lib/utils/helper';
    import Records from '$lib/Records/index.svelte';

    let loading = true;
    let error = '';

    let leagueData = null;
    let totals = null;
    let stale = false;
    let leagueTeamManagers = null;

    onMount(async () => {
        try {
            const [recordsData, transactionData, managersData] = await Promise.all([
                getLeagueRecords(),
                getLeagueTransactions(false),
                getLeagueTeamManagers()
            ]);

            leagueData = recordsData;
            totals = transactionData?.totals || {};
            stale = transactionData?.stale || false;
            leagueTeamManagers = managersData;
        } catch (e) {
            error = e?.message || 'Unable to load league records.';
        } finally {
            loading = false;
        }
    });
</script>

{#if loading}
    <div class="loading-page">
        <div class="loading-card">
            <div class="loading-wizard">✨</div>
            <div>Gathering the records...</div>
        </div>
    </div>
{:else if error}
    <div class="loading-page">
        <div class="loading-card">
            <h2>Something went wrong</h2>
            <p>{error}</p>
        </div>
    </div>
{:else if leagueData && totals && leagueTeamManagers}
    <Records {leagueData} {totals} {stale} {leagueTeamManagers} />
{/if}

<style>
    .loading-page {
        min-height: 70vh;
        display: grid;
        place-items: center;
        padding: 30px;
    }

    .loading-card {
        width: min(92%, 520px);
        padding: 28px;
        border-radius: 22px;
        background: rgba(255,255,255,0.96);
        border: 1px solid rgba(73,126,213,0.16);
        box-shadow: 0 10px 30px rgba(30,64,175,0.10);
        text-align: center;
        color: #334155;
    }

    .loading-wizard {
        font-size: 32px;
        margin-bottom: 10px;
    }

    .loading-card h2 {
        margin: 0 0 8px;
        color: #6d28d9;
    }

    .loading-card p {
        margin: 0;
    }
</style>
