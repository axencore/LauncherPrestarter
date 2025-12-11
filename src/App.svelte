<script lang="ts">
    import "reset-css";
    import { invoke } from "@tauri-apps/api/core";
    import { listen } from "@tauri-apps/api/event";
    import { getCurrentWindow } from "@tauri-apps/api/window";
    import { onMount } from "svelte";

    import "$lib/assets/css/global.scss";
    import StarsAnimation from "$lib/components/StarsAnimation.svelte";

    import { appConfig, tauriCommands, tauriEvents } from "$lib/config/app";
    import { DownloadTracker } from "$lib/utils/download";
    import type {
        ExtractProgressEvent,
        DownloadProgressEvent,
    } from "$lib/types/events";
    import DownloadBlock from "$lib/components/ui/DownloadBlock.svelte";

    const defaultStatus = "СКАЧИВАЕМ JAVA ДЛЯ ИГРЫ";

    let lastSpeedUpdate = 0;
    let error = "";
    let speedMb: string = "";
    let percentage: number = 0;
    let totalLabel = "";
    let done = false;
    let running = false;
    let statusText = defaultStatus;

    const appWindow = getCurrentWindow();
    const downloadTracker = new DownloadTracker();

    const minimize = () => appWindow.minimize();
    const close_application = () => invoke(tauriCommands.closeApp);

    async function startDownload() {
        try {
            error = "";
            done = false;
            running = false;
            downloadTracker.reset();
            speedMb = "";

            const result = await invoke<string>(tauriCommands.startDownload);
            console.log("Результат запуска:", result || "успешно");
        } catch (err) {
            console.error("Ошибка инициализации загрузки:", err);
            error = String(err);
            speedMb = "ERR";
        }
    }

    async function setupListeners() {
        try {
            await listen<DownloadProgressEvent>(
                tauriEvents.downloadProgress,
                (event) => {
                    const now = Date.now();
                    const current = event.payload.downloaded;
                    const total = event.payload.total;

                    const result = downloadTracker.update(current, total);

                    speedMb = result.speed;
                    percentage = result.percentage;

                    if (now - lastSpeedUpdate > 200) {
                        totalLabel = downloadTracker.formatTotalLabel(total);
                        lastSpeedUpdate = now;
                    }
                },
            );

            await listen<ExtractProgressEvent>(
                tauriEvents.extractProgress,
                (event) => {
                    speedMb = "--";
                    percentage = downloadTracker.percentageCalculation(
                        event.payload.processed,
                        event.payload.total,
                    );
                    totalLabel = downloadTracker.formatTotalLabel(
                        event.payload.total,
                    );
                },
            );
            await listen<string>(tauriEvents.error, (event) => {
                speedMb = "ERR";
                error = event.payload;
            });
            await listen(tauriEvents.running, () => {
                running = true;
            });
            await listen(tauriEvents.done, () => {
                done = true;
                close_application();
            });
        } catch (err) {
            console.error("Failed to setup listeners:", err);
        }
    }

    $: statusText = error
        ? error
        : done
          ? "ГОТОВО"
          : running
            ? "УСТАНОВКА ФАЙЛОВ"
            : defaultStatus;

    onMount(() => {
        setupListeners();
        setTimeout(startDownload, appConfig.download.initialDelay);
    });
</script>

<svelte:head>
    <title>EncoreCraft Java Downloader</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<div data-tauri-drag-region class="app">
    <div class="frame" data-tauri-drag-region>
        <div class="background" aria-hidden="true">
            <div class="gradient"></div>
            <div class="pattern-mask">
                <div class="pattern">
                    <StarsAnimation />
                </div>
            </div>
            <div class="grain"></div>
        </div>

        <header class="titlebar">
            <div class="controls">
                <button class="control" on:click={minimize} aria-label="Свернуть">
                    <span class="icon minus"></span>
                </button>
                <button class="control" on:click={close_application} aria-label="Закрыть">
                    <span class="icon close"></span>
                </button>
            </div>
        </header>

        <main class="hero" data-tauri-drag-region>
            <div class="logo-wrap" data-tauri-drag-region>
                <div class="logo-image" aria-label="Логотип"></div>
            </div>
        </main>

        <section class="loading" data-tauri-drag-region>
            <div class="download-area">
                <DownloadBlock
                    {error}
                    {speedMb}
                    {percentage}
                    totalLabel={totalLabel}
                    statusLabel={statusText}
                />
            </div>
        </section>
    </div>
</div>

<style lang="scss">
    :global(.app) {
        width: 100vw;
        height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
        overflow: hidden;
        background: transparent;
        font-family: "Montserrat", "Open Sans", sans-serif;
        padding: 0;
    }

    .frame {
        position: relative;
        width: 100%;
        height: 100%;
        padding: 20px 20px 30px;
        display: flex;
        flex-direction: column;
        gap: 22px;
        border-radius: 15px;
        overflow: hidden;
        box-shadow: 0 16px 45px rgba(0, 0, 0, 0.45);
        isolation: isolate;
    }

    .background {
        position: absolute;
        inset: 0;
        border-radius: 15px;
        overflow: hidden;
        pointer-events: none;
        z-index: 0;
    }

    .frame > :not(.background) {
        position: relative;
        z-index: 1;
    }

    .background .gradient {
        position: absolute;
        inset: 0;
        background: linear-gradient(108deg, #263721 0%, #1e1531 100%);
        overflow: hidden;
    }

    .background .gradient::after {
        content: "";
        position: absolute;
        inset: 0;
        background: url("/background-image.png") -251.032px -169.245px / 182.253% 174.668% no-repeat;
        opacity: 0.5;
        pointer-events: none;
    }

    .background .pattern-mask {
        position: absolute;
        width: 1120px;
        height: 1120px;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        display: flex;
        justify-content: center;
        align-items: center;
        pointer-events: none;
        overflow: visible;
        isolation: isolate;
    }

    .background .pattern {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%) rotate(0deg);
        width: 1120px;
        height: 1120px;
        mask-image: url("/Rectangle%2010.png");
        -webkit-mask-image: url("/Rectangle%2010.png");
        mask-repeat: no-repeat;
        -webkit-mask-repeat: no-repeat;
        mask-position: center;
        -webkit-mask-position: center;
        mask-size: 1120px 1120px;
        -webkit-mask-size: 1120px 1120px;
        mask-type: alpha;
        pointer-events: none;
        mix-blend-mode: screen;
        filter: brightness(1.1);
    }

    .background .grain {
        position: absolute;
        inset: 0;
        background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160' viewBox='0 0 160 160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.8' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.18'/%3E%3C/svg%3E");
        mix-blend-mode: soft-light;
        opacity: 0.4;
    }

    .titlebar {
        display: flex;
        justify-content: flex-end;
        align-items: center;
    }

    .controls {
        display: flex;
        gap: 11px;
    }

    .control {
        appearance: none;
        border: 3px solid rgba(255, 255, 255, 0.1);
        border-radius: 7px;
        width: 36px;
        height: 36px;
        padding: 8px;
        display: inline-flex;
        justify-content: center;
        align-items: center;
        background: rgba(20, 16, 25, 0.45);
        backdrop-filter: blur(7.5px);
        cursor: pointer;
        transition: transform 0.12s ease, border-color 0.12s ease, box-shadow 0.12s ease;
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.25);

        &:hover {
            border-color: rgba(196, 172, 124, 0.5);
            transform: translateY(-1px);
            box-shadow: 0 14px 30px rgba(0, 0, 0, 0.35);
        }
    }

    .icon {
        position: relative;
        display: block;
        width: 14px;
        height: 14px;
    }

    .icon::before,
    .icon::after {
        content: "";
        position: absolute;
        inset: 0;
        margin: auto;
        background: #b89a61;
        border-radius: 999px;
        box-shadow: 0 0 15px rgba(184, 154, 97, 0.4);
    }

    .icon.minus::before {
        width: 14px;
        height: 2.6px;
        top: 50%;
        transform: translateY(-50%);
    }

    .icon.minus::after {
        display: none;
    }

    .icon.close::before,
    .icon.close::after {
        width: 14px;
        height: 2.6px;
        top: 50%;
        transform-origin: center;
        transform: translateY(-50%) rotate(45deg);
    }

    .icon.close::after {
        transform: translateY(-50%) rotate(-45deg);
    }

    .hero {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 20px;
    }

    .logo-wrap {
        position: relative;
        width: min(220px, 70vw);
        height: min(220px, 70vw);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .logo-image {
        width: 178px;
        height: 178px;
        background: url("/Логотип%20Существо%20Биг%201.png") center/cover no-repeat;
        filter: drop-shadow(0 10px 24px rgba(0, 0, 0, 0.6));
    }

    .loading {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 17px;
    }

    .download-area {
        width: min(720px, 100%);
    }

</style>
