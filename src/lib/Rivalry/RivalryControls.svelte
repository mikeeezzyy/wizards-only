<script>
    import IconButton from '@smui/icon-button';

    export let year, displayWeek, selected, length;

    const nav = (dir) => {
        if (dir === "left") {
            if (selected > 0) {
                selected--;
            } else {
                selected = length - 1;
            }
        } else {
            if (selected < length - 1) {
                selected++;
            } else {
                selected = 0;
            }
        }
    };
</script>

<style>
    .container {
        display: flex;
        justify-content: center;
        align-items: center;
        width: fit-content;
        margin: 18px auto 8px;
        padding: 8px 14px;
        border-radius: 18px;
        background: rgba(255, 255, 255, 0.92);
        border: 1px solid rgba(37, 99, 235, 0.12);
        box-shadow: 0 6px 20px rgba(30, 64, 175, 0.08);
    }

    h4 {
        display: inline-block;
        text-align: center;
        font-family: "Luckiest Guy", cursive;
        font-size: 1.4rem;
        line-height: 1.2;
        color: var(--wizard-purple-dark, #35116b);
        margin: 4px 18px;
        min-width: 170px;
    }

    :global(.material-icons) {
        color: var(--wizard-purple, #6d28d9);
        border-radius: 50%;
        transition: all 0.15s ease;
    }

    :global(.material-icons:hover) {
        background: var(--wizard-blue-light, #ddf7ff);
        color: var(--wizard-blue, #29b6f6);
    }

    @media (max-width: 650px) {
        .container {
            width: 90%;
            box-sizing: border-box;
        }

        h4 {
            font-size: 1.15rem;
            margin: 4px 10px;
            min-width: 140px;
        }
    }

    @media (max-width: 400px) {
        h4 {
            font-size: 1rem;
            min-width: 120px;
        }
    }
</style>

<div class="container">
    {#if length > 0}
        <IconButton
            class="material-icons"
            onclick={() => nav("left")}
            aria-label="Previous matchup"
        >
            chevron_left
        </IconButton>
    {/if}

    <h4>{year} Week {displayWeek}</h4>

    {#if length > 0}
        <IconButton
            class="material-icons"
            onclick={() => nav("right")}
            aria-label="Next matchup"
        >
            chevron_right
        </IconButton>
    {/if}
</div>
