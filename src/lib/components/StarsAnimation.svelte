<script lang="ts">
    import swirlSvg from "../../stars-animation.txt?raw";
</script>

<div class="stars" aria-hidden="true">
    {@html swirlSvg}
</div>

<style>
    .stars {
        position: absolute;
        inset: 0;
        pointer-events: none;
        display: flex;
        justify-content: center;
        align-items: center;
        mix-blend-mode: screen;
        filter: brightness(1.05);
        opacity: 0.9;
    }

    .stars :global(svg) {
        width: 1120px;
        height: 1120px;
        animation: rotate-steps 32000ms linear infinite;
    }

    @keyframes rotate-steps {
        0% {
            transform: rotate(0deg);
        }
        12.5% {
            transform: rotate(45deg);
        }
        25% {
            transform: rotate(90deg);
        }
        37.5% {
            transform: rotate(135deg);
        }
        50% {
            transform: rotate(180deg);
        }
        62.5% {
            transform: rotate(225deg);
        }
        75% {
            transform: rotate(270deg);
        }
        87.5% {
            transform: rotate(315deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    @media (prefers-reduced-motion: reduce) {
        .stars :global(svg) {
            animation-duration: 0.01ms;
            animation-iteration-count: 1;
        }
    }
</style>
