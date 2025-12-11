<script lang="ts">
    import ProgressBar from "./ProgressBar.svelte";

    export let error = "";
    export let speedMb: string | number;
    export let percentage: number;
    export let totalLabel = "";
    export let statusLabel = "СКАЧИВАЕМ JAVA ДЛЯ ИГРЫ";
    export let showMeta = false;

    $: speedLabel =
        speedMb && speedMb !== "ERR" && speedMb !== "--" && speedMb !== ""
            ? `${speedMb} Mbps`
            : "";

    $: metaLabel =
        error || (!totalLabel && percentage === 0)
            ? ""
            : totalLabel
              ? `${Math.min(Math.round(percentage), 100)}% · ${totalLabel}${speedLabel ? ` · ${speedLabel}` : ""}`
              : `${Math.min(Math.round(percentage), 100)}%${speedLabel ? ` · ${speedLabel}` : ""}`;
</script>

<div data-tauri-drag-region class="download-block">
    <p class:errored={!!error} data-tauri-drag-region>{statusLabel}</p>
    <ProgressBar class={error ? "errored" : ""} {percentage} />
    {#if showMeta && metaLabel}
        <small data-tauri-drag-region>{metaLabel}</small>
    {/if}
</div>

<style lang="scss">
    .download-block {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 12px;
        text-align: center;
        width: 100%;
    }

    p {
        font-size: 14px;
        font-weight: 600;
        letter-spacing: 2.7px;
        text-transform: uppercase;
        color: #ffffff;
        margin: 0;
    }

    p.errored {
        color: $error;
    }

    small {
        font-size: 0.8rem;
        color: $text-secondary;
        letter-spacing: 0.5px;
    }

</style>
