<script>
    import { goto } from "$app/navigation";
    import { getTeamData } from "$lib/utils/helperFunctions/universalFunctions";

    export let playerOne, playerTwo, leagueTeamManagers;

    const users = Object.keys(leagueTeamManagers.users);

    $: usersOne = users.filter((u) => u !== playerTwo);
    $: usersTwo = users.filter((u) => u !== playerOne);

    const analyzeRivalry = (p1, p2) => {
        if (!p1 || !p2) {
            return;
        }

        goto(
            `/rivalry?player_one=${p1}&player_two=${p2}`,
            {
                noscroll: true,
                keepfocus: true
            }
        );
    };

    $: analyzeRivalry(playerOne, playerTwo);
</script>

<style>
    .selectors {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 28px;
        width: 100%;
        max-width: 950px;
        margin: 10px auto 18px;
    }

    .manager {
        flex: 1;
        max-width: 330px;
        text-align: center;
    }

    .container {
        position: relative;
        width: 100%;
    }

    .selectInput {
        box-sizing: border-box;
        width: 100%;
        min-height: 58px;
        padding: 12px 48px;
        border-radius: 18px;
        font-family: "Roboto", sans-serif;
        font-size: 1rem;
        font-weight: 700;
        text-align: center;
        color: var(--wizard-text, #10182f);
        background-color: rgba(255, 255, 255, 0.96);
        appearance: none !important;
        -webkit-appearance: none !important;
        -moz-appearance: none !important;
        background-image: url("/dropdown.png");
        background-repeat: no-repeat;
        background-size: 18px;
        box-shadow: 0 8px 24px rgba(30, 64, 175, 0.1);
        transition:
            border-color 0.15s ease,
            box-shadow 0.15s ease,
            transform 0.15s ease;
        cursor: pointer;
    }

    .selectInput:hover {
        transform: translateY(-1px);
        box-shadow: 0 10px 28px rgba(30, 64, 175, 0.14);
    }

    .selectInput:focus {
        outline: none;
        transform: translateY(-1px);
    }

    .left {
        border: 2px solid var(--wizard-purple, #6d28d9);
        background-position: right 16px center;
        text-align: left;
        padding-left: 54px;
    }

    select.left:focus {
        border-color: var(--wizard-blue, #29b6f6);
        box-shadow: 0 0 0 4px rgba(41, 182, 246, 0.12);
    }

    .right {
        border: 2px solid var(--wizard-blue, #29b6f6);
        background-position: right 16px center;
        text-align: right;
        padding-right: 54px;
    }

    select.right:focus {
        border-color: var(--wizard-purple, #6d28d9);
        box-shadow: 0 0 0 4px rgba(109, 40, 217, 0.12);
    }

    select option {
        text-align: left;
    }

    .vs {
        flex: 0 0 auto;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 54px;
        height: 54px;
        border-radius: 50%;
        background: linear-gradient(
            135deg,
            var(--wizard-purple, #6d28d9),
            var(--wizard-blue, #29b6f6)
        );
        color: white;
        font-family: "Permanent Marker", cursive;
        font-size: 1.35rem;
        box-shadow: 0 8px 22px rgba(109, 40, 217, 0.22);
        z-index: 2;
    }

    .avatar {
        position: absolute;
        top: 50%;
        width: 62px;
        height: 62px;
        object-fit: cover;
        transform: translateY(-50%);
        border-radius: 50%;
        background: white;
        box-shadow: 0 6px 18px rgba(30, 64, 175, 0.18);
        z-index: 3;
    }

    .avatarLeft {
        left: -31px;
        border: 4px solid var(--wizard-purple, #6d28d9);
    }

    .avatarRight {
        right: -31px;
        border: 4px solid var(--wizard-blue, #29b6f6);
    }

    @media (max-width: 700px) {
        .selectors {
            gap: 14px;
        }

        .selectInput {
            min-height: 52px;
            padding-left: 44px;
            padding-right: 44px;
            font-size: 0.9rem;
        }

        .avatar {
            width: 48px;
            height: 48px;
        }

        .avatarLeft {
            left: -18px;
        }

        .avatarRight {
            right: -18px;
        }

        .vs {
            width: 46px;
            height: 46px;
            font-size: 1.1rem;
        }
    }

    @media (max-width: 530px) {
        .selectors {
            flex-direction: column;
            gap: 12px;
        }

        .manager {
            width: 90%;
            max-width: 360px;
        }

        .vs {
            width: 44px;
            height: 44px;
            font-size: 1rem;
        }

        .avatarLeft {
            left: -12px;
        }

        .avatarRight {
            right: -12px;
        }
    }
</style>

<div class="selectors">
    <!-- Manager 1 -->
    <div class="manager">
        <div class="container">
            <select
                class="selectInput left"
                id="managerOne"
                name="managerOne"
                bind:value={playerOne}
            >
                <option value={null}>Select a manager</option>

                {#each usersOne as user}
                    <option value={user}>
                        {leagueTeamManagers.users[user].display_name}
                    </option>
                {/each}
            </select>

            {#if playerOne}
                <img
                    class="avatar avatarLeft"
                    src={getTeamData(leagueTeamManagers.users, playerOne).avatar}
                    alt="Manager one avatar"
                />
            {/if}
        </div>
    </div>

    <!-- VS -->
    <span class="vs">VS</span>

    <!-- Manager 2 -->
    <div class="manager">
        <div class="container">
            <select
                class="selectInput right"
                id="managerTwo"
                name="managerTwo"
                bind:value={playerTwo}
            >
                <option value={null}>Select a manager</option>

                {#each usersTwo as user}
                    <option value={user}>
                        {leagueTeamManagers.users[user].display_name}
                    </option>
                {/each}
            </select>

            {#if playerTwo}
                <img
                    class="avatar avatarRight"
                    src={getTeamData(leagueTeamManagers.users, playerTwo).avatar}
                    alt="Manager two avatar"
                />
            {/if}
        </div>
    </div>
</div>
