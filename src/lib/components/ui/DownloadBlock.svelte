<script lang="ts">
    import ProgressBar from "./ProgressBar.svelte";

    export let error = "";
    export let speedMb: string | number;
    export let percentage: number;
    export let totalLabel = "";
    export let filesLabel = "";
    export let statusLabel = "Downloading JAVA";
    export let showMeta = false;

    $: speedLabel =
        speedMb === "ERR"
            ? "ERR"
            : speedMb && speedMb !== "--" && speedMb !== ""
              ? `${speedMb} Mbps`
              : "";

    const filesPlaceholder = "—";
    $: placeholder = speedMb === "--" ? "Распаковка" : "—";
    $: filesStat = filesLabel || filesPlaceholder;
    $: speedStat = speedLabel || (error ? "" : placeholder);

    $: metaLabel =
        error || (!totalLabel && percentage === 0)
            ? ""
            : totalLabel
              ? `${Math.min(Math.round(percentage), 100)}% - ${totalLabel}${speedLabel ? ` - ${speedLabel}` : ""}`
              : `${Math.min(Math.round(percentage), 100)}%${speedLabel ? ` - ${speedLabel}` : ""}`;
</script>

<div data-tauri-drag-region class="download-block">
    <div class="stats" data-tauri-drag-region>
        <span class="stat files">{filesStat}</span>
        <p class:errored={!!error}>{statusLabel}</p>
        <span class="stat speed">{speedStat}</span>
    </div>
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

    .stats {
        width: 100%;
        display: grid;
        grid-template-columns: 1fr auto 1fr;
        align-items: center;
        gap: 12px;
    }

    .stat {
        font-size: 12px;
        font-weight: 400;
        letter-spacing: 0.5px;
        color: $text-description;
        opacity: 0.9;
    }

    .stat.files {
        text-align: left;
    }

    .stat.speed {
        text-align: right;
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
