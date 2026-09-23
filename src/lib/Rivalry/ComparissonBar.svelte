<script>
    export let sideOne, sideTwo, label, unit;
</script>

<style>
    .comparison {
        width: 92%;
        max-width: 850px;
        margin: 24px auto 30px;
    }

    h4 {
        text-align: center;
        font-family: "Luckiest Guy", cursive;
        font-size: 1.45rem;
        font-weight: 400;
        letter-spacing: 0.4px;
        color: var(--wizard-purple-dark, #35116b);
        margin: 0 0 12px;
    }

    .enclosure {
        padding: 4px 8px 8px;
        text-align: center;
    }

    .bar {
        position: relative;
        width: 100%;
        height: 14px;
        overflow: hidden;
        border-radius: 999px;
        background: #edf5fc;
        border: 1px solid rgba(37, 99, 235, 0.14);
        box-shadow: inset 0 1px 3px rgba(30, 64, 175, 0.08);
    }

    .bar::after {
        content: "";
        position: absolute;
        top: 0;
        bottom: 0;
        left: 50%;
        width: 2px;
        transform: translateX(-1px);
        background: rgba(255, 255, 255, 0.92);
        z-index: 3;
    }

    .side {
        position: absolute;
        top: 0;
        height: 100%;
        transition: width 0.25s ease;
    }

    .side_one {
        right: 50%;
        background: linear-gradient(90deg, #6d28d9, #9b5de5);
        border-radius: 999px 0 0 999px;
    }

    .side_two {
        left: 50%;
        background: linear-gradient(90deg, #29b6f6, #00cfe8);
        border-radius: 0 999px 999px 0;
    }

    .stats {
        display: flex;
        width: 100%;
        margin: 9px auto 0;
        justify-content: space-between;
        align-items: center;
        font-size: 1rem;
        font-weight: 700;
        color: var(--wizard-muted, #64748b);
    }

    .stat {
        min-width: 90px;
    }

    .oneWinner {
        color: var(--wizard-purple, #6d28d9);
        font-weight: 900;
    }

    .twoWinner {
        color: #0796c9;
        font-weight: 900;
    }

    @media (max-width: 650px) {
        h4 {
            font-size: 1.15rem;
        }

        .stats {
            font-size: 0.9rem;
        }
    }

    @media (max-width: 400px) {
        h4 {
            font-size: 1.05rem;
        }

        .stats {
            font-size: 0.85rem;
        }
    }
</style>

<div class="comparison">
    <h4>{label}</h4>

    <div class="enclosure">
        <div class="bar">
            <div
                class="side side_one"
                style="width: {sideOne + sideTwo > 0 ? (sideOne / (sideOne + sideTwo)) * 50 : 0}%"
            ></div>

            <div
                class="side side_two"
                style="width: {sideOne + sideTwo > 0 ? (sideTwo / (sideOne + sideTwo)) * 50 : 0}%"
            ></div>
        </div>

        <div class="stats">
            <span class="stat{sideOne > sideTwo ? ' oneWinner' : ''}">
                {sideOne} {unit}
            </span>

            <span class="stat{sideTwo > sideOne ? ' twoWinner' : ''}">
                {sideTwo} {unit}
            </span>
        </div>
    </div>
</div>
