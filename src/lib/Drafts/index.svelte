<script>
    import { waitForAll } from '$lib/utils/helper';
    import LinearProgress from '@smui/linear-progress';
    import Draft from './Draft.svelte';

    export let upcomingDraftData, previousDraftsData, leagueTeamManagersData, playersData;
</script>

<div class="drafts-page">
    <div class="drafts-card">

        <div class="page-heading">
            <div class="heading-left">
                <span class="heading-icon">🏈</span>
                <h1>DRAFTS</h1>
            </div>
        </div>

        <div class="season-bar">
            <span>WIZARDS ONLY</span>
            <span>DRAFT HISTORY</span>
        </div>

        <section class="draft-section">
            <div class="section-heading">
                <span class="section-icon">🏈</span>
                <div>
                    <h2>UPCOMING DRAFT</h2>
                    <p>Your next draft</p>
                </div>
            </div>

            {#await waitForAll(upcomingDraftData, leagueTeamManagersData, playersData)}
                <div class="loading">
                    <p>Retrieving upcoming draft...</p>
                    <LinearProgress indeterminate />
                </div>
            {:then [upcomingDraft, leagueTeamManagers, {players}]}
                <div class="draft-title">{upcomingDraft.year} DRAFT</div>
                <Draft draftData={upcomingDraft} {leagueTeamManagers} year={upcomingDraft.year} {players} />
            {:catch error}
                <div class="error">Something went wrong: {error.message}</div>
            {/await}
        </section>

        {#await waitForAll(previousDraftsData, leagueTeamManagersData, playersData)}
            <section class="draft-section">
                <div class="section-heading">
                    <span class="section-icon">📜</span>
                    <div>
                        <h2>PREVIOUS DRAFTS</h2>
                        <p>Draft history</p>
                    </div>
                </div>

                <div class="loading">
                    <p>Retrieving previous drafts...</p>
                    <LinearProgress indeterminate />
                </div>
            </section>
        {:then [previousDrafts, leagueTeamManagers, {players}]}
            {#if previousDrafts.length}
                <section class="draft-section">
                    <div class="section-heading">
                        <span class="section-icon">📜</span>
                        <div>
                            <h2>PREVIOUS DRAFTS</h2>
                            <p>Every completed Wizards Only draft</p>
                        </div>
                    </div>

                    {#each previousDrafts as previousDraft}
                        <div class="draft-title">{previousDraft.year} DRAFT</div>
                        <Draft draftData={previousDraft} previous={true} {leagueTeamManagers} year={previousDraft.year} {players} />
                    {/each}
                </section>
            {/if}
        {:catch error}
            <section class="draft-section">
                <div class="error">Something went wrong: {error.message}</div>
            </section>
        {/await}

    </div>
</div>

<style>
    :global(body) {
        font-family: 'Poppins', sans-serif;
    }

    .drafts-page {
        width: 100%;
        min-height: 100vh;
        padding: 28px 24px 60px;
    }

    .drafts-card {
        width: 100%;
        max-width: 1200px;
        margin: 0 auto;
        background: linear-gradient(
            135deg,
            rgba(255, 255, 255, 0.96),
            rgba(240, 247, 255, 0.92)
        );
        border-radius: 20px;
        border-top: 5px solid #7a50dc;
        box-shadow: 0 8px 28px rgba(44, 69, 145, 0.16);
        overflow: hidden;
    }

    .page-heading {
        display: flex;
        align-items: center;
        min-height: 70px;
        padding: 12px 22px;
        border-bottom: 1px solid rgba(80, 110, 180, 0.15);
    }

    .heading-left {
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .heading-icon {
        font-size: 24px;
    }

    h1 {
        margin: 0;
        font-family: 'Luckiest Guy', cursive;
        font-size: 28px;
        line-height: 1;
        color: #5b189c;
    }

    .season-bar {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 11px 22px;
        background: rgba(236, 245, 255, 0.65);
        border-bottom: 1px solid rgba(80, 110, 180, 0.12);
        font-size: 9px;
        font-weight: 800;
        letter-spacing: 0.8px;
        color: #6274a7;
    }

    .draft-section {
        margin: 18px;
        padding: 0;
        border: 1px solid rgba(85, 122, 194, 0.18);
        border-radius: 18px;
        background: rgba(247, 251, 255, 0.74);
        box-shadow: 0 3px 12px rgba(50, 85, 150, 0.05);
        overflow: hidden;
    }

    .section-heading {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 15px 18px;
        border-bottom: 1px solid rgba(85, 122, 194, 0.14);
        background: rgba(255, 255, 255, 0.62);
    }

    .section-icon {
        font-size: 19px;
    }

    .section-heading h2 {
        margin: 0;
        font-family: 'Luckiest Guy', cursive;
        font-size: 20px;
        line-height: 1;
        color: #5b189c;
    }

    .section-heading p {
        margin: 4px 0 0;
        font-size: 9px;
        font-weight: 600;
        color: #7c8baa;
    }

    .draft-title {
        margin: 16px 16px 10px;
        font-family: 'Luckiest Guy', cursive;
        font-size: 17px;
        color: #2858bd;
        letter-spacing: 0.3px;
    }

    .loading {
        width: min(92%, 500px);
        margin: 26px auto 30px;
        padding: 20px;
        border-radius: 14px;
        background: rgba(255, 255, 255, 0.72);
        text-align: center;
        color: #6274a7;
        font-size: 11px;
        font-weight: 600;
    }

    .loading p {
        margin: 0 0 12px;
    }

    .error {
        margin: 18px;
        padding: 16px;
        border-radius: 12px;
        background: rgba(255, 245, 247, 0.9);
        color: #8c3f56;
        font-size: 12px;
        font-weight: 600;
    }

    @media (max-width: 700px) {
        .drafts-page {
            padding: 16px 8px 30px;
        }

        .page-heading {
            min-height: 60px;
            padding: 10px 14px;
        }

        h1 {
            font-size: 22px;
        }

        .season-bar {
            padding: 9px 14px;
        }

        .draft-section {
            margin: 10px 8px;
            border-radius: 15px;
        }

        .section-heading {
            padding: 13px 14px;
        }

        .section-heading h2 {
            font-size: 16px;
        }

        .draft-title {
            margin: 13px 10px 8px;
            font-size: 15px;
        }
    }
</style>
