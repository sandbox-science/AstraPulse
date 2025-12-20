<script>
    import { onMount } from "svelte";
    import About from './components/About.svelte';
    let showAbout = false;

    let duration  = 25 * 60 * 1000; // 25 minutes default
    let remaining = duration;
    let running   = false;
    let lastTime  = 0;
    let rafId     = null;

    // UI state
    let customMinutes = 25;
    const presetButtons = [
        { label: "15", ms: 15 * 60 * 1000 },
        { label: "25", ms: 25 * 60 * 1000 },
        { label: "50", ms: 50 * 60 * 1000 },
    ];

    function format(ms)
    {
        const total = Math.ceil(ms / 1000);
        const m     = Math.floor(total / 60);
        const s     = total % 60;
        return `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
    }

    function progress()
    {
        if (!duration)
        {
            return 0;
        }
        return Math.min(1, Math.max(0, 1 - remaining / duration));
    }

    function start()
    {
        if (!duration)
        {
            return;
        }
        running  = true;
        lastTime = performance.now();
        rafId    = requestAnimationFrame(tick);
    }

    function stop()
    {
        running = false;
        if (rafId)
        {
            cancelAnimationFrame(rafId);
        }
        rafId = null;
    }

    function reset()
    {
        stop();
        remaining = duration;
    }

    function tick(now)
    {
        if (!duration)
        {
            return;
        }
        const delta = now - lastTime;
        remaining   = Math.max(0, remaining - delta);
        lastTime    = now;
        
        if (remaining <= 0)
        {
            stop();
            onTimerEnd();
        }
        else
        {
            rafId = requestAnimationFrame(tick);
        }
    }

    function setPreset(ms)
    {
        duration      = ms;
        customMinutes = Math.round(ms / 60000);
        reset();
    }

    function setCustom()
    {
        const mins = Number(customMinutes);
        if (!Number.isFinite(mins) || mins <= 0)
        {
            return;
        }
        duration = Math.round(mins * 60 * 1000);
        reset();
    }

    function playSound()
    {
        try
        {
            const ctx         = new AudioContext();
            const o           = ctx.createOscillator();
            const g           = ctx.createGain();
            o.type            = "sine";
            o.frequency.value = 880;
            g.gain.value      = 0.08;
            o.connect(g);
            g.connect(ctx.destination);
            o.start();
            setTimeout(() => {
                o.stop();
                ctx.close();
            }, 800);
        }
        catch (e)
        {
            console.warn("Audio failed", e);
        }
    }

    async function notify(title, body)
    {
        try
        {
            const { Notification } = await import(
                "@tauri-apps/api/notification"
            );
            new Notification({ title, body }).show();
        }
        catch (e)
        {
            try
            {
                if (
                    window.Notification &&
                    Notification.permission !== "granted"
                )
                {
                    await window.Notification.requestPermission();
                }
                if (
                    window.Notification &&
                    Notification.permission === "granted"
                )
                {
                    new window.Notification(title, { body });
                }
            }
            catch (err)
            {
                console.warn("Notification failed", err);
            }
        }
    }

    function onTimerEnd()
    {
        playSound();
        notify("Timer finished", "Take a break!");
    }

    onMount(() => {
        remaining = duration;
        customMinutes = Math.round(duration / 60000);
    });
</script>

<div class="shell">
    <div class="card" role="application" aria-label="AstraPulse timer">
        <header class="top">
            <div class="brand">
                <div class="dot" aria-hidden="true"></div>
                <div class="titles">
                    <div class="title">AstraPulse</div>
                    <div class="subtitle">{running ? "Running" : "Ready"}</div>
                </div>
            </div>

            <div class="header-actions">
                <button
                    class="icon secondary"
                    type="button"
                    on:click={reset}
                    aria-label="Reset"
                    title="Reset"
                >
                    ↺
                </button>

                <button
                    class="icon secondary"
                    type="button"
                    on:click={() => (showAbout = true)}
                    aria-label="About"
                    title="About"
                >
                    ℹ
                </button>
            </div>
        </header>

        <section class="timer">
            <div class="ring" style={`--p:${progress() * 100}%`}>
                <div class="time" aria-live="polite">{format(remaining)}</div>
            </div>
        </section>

        <section class="actions">
            <button
                class="primary"
                type="button"
                on:click={start}
                disabled={running}
            >
                Start
            </button>
            <button
                class="secondary"
                type="button"
                on:click={stop}
                disabled={!running}
            >
                Pause
            </button>
            <button class="ghost" type="button" on:click={reset}>
                Reset
            </button>
        </section>

        <section class="panel">
            <div class="panelTitle">Presets</div>
            <div class="presets" role="group" aria-label="Preset durations">
                {#each presetButtons as p}
                    <button
                        type="button"
                        class="pill"
                        class:active={duration === p.ms}
                        on:click={() => setPreset(p.ms)}
                    >
                        {p.label}m
                    </button>
                {/each}
            </div>

            <div class="panelTitle">Custom</div>
            <div class="custom">
                <label class="field">
                    <span class="label">Minutes</span>
                    <input
                        type="number"
                        min="1"
                        step="1"
                        bind:value={customMinutes}
                        inputmode="numeric"
                        placeholder="25"
                    />
                </label>
                <button type="button" class="secondary" on:click={setCustom}
                    >Set</button
                >
            </div>

            <div class="hint">
                Tip: keep the window in the background, notifications still fire
                when the timer ends.
            </div>
        </section>
    </div>
    <About bind:open={showAbout} />
</div>

<style>
    :global(body) {
        margin: 0;
        min-height: 100vh;
        display: grid;
        place-items: center;
        font-family:
            ui-sans-serif,
            system-ui,
            -apple-system,
            Segoe UI,
            Roboto,
            Helvetica Neue,
            Arial,
            Noto Sans,
            "Apple Color Emoji",
            "Segoe UI Emoji";
        color: #0f172a;
        background: radial-gradient(
                1200px 800px at 20% 10%,
                rgba(99, 102, 241, 0.12),
                transparent 60%
            ),
            radial-gradient(
                900px 600px at 80% 30%,
                rgba(14, 165, 233, 0.1),
                transparent 55%
            ),
            linear-gradient(180deg, #f8fafc, #ffffff);
    }

    .shell {
        padding: 24px;
        width: min(440px, 100%);
    }

    .card {
        background: rgba(255, 255, 255, 0.7);
        backdrop-filter: blur(12px);
        border: 1px solid rgba(148, 163, 184, 0.25);
        border-radius: 20px;
        box-shadow:
            0 18px 50px rgba(2, 6, 23, 0.1),
            0 2px 8px rgba(2, 6, 23, 0.06);
        padding: 18px;
    }

    .top {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 12px;
        margin-bottom: 12px;
    }

    .header-actions {
        display: flex;
        gap: 8px;
        align-items: center;
    }

    .brand {
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .dot {
        width: 12px;
        height: 12px;
        border-radius: 999px;
        background: linear-gradient(135deg, #6366f1, #0ea5e9);
        box-shadow: 0 0 0 6px rgba(99, 102, 241, 0.12);
    }

    .titles .title {
        font-weight: 650;
        letter-spacing: -0.02em;
        font-size: 14px;
        line-height: 1.2;
    }

    .titles .subtitle {
        font-size: 12px;
        color: #475569;
        margin-top: 2px;
    }

    .timer {
        display: grid;
        place-items: center;
        padding: 14px 0 8px;
    }

    /* Progress ring (conic gradient) */
    .ring {
        --p: 0%;
        width: 240px;
        height: 240px;
        border-radius: 999px;
        display: grid;
        place-items: center;
        background: conic-gradient(
            from 270deg,
            rgba(99, 102, 241, 0.95) var(--p),
            rgba(148, 163, 184, 0.413) 0
        );
        position: relative;
        box-shadow: 0 16px 30px rgba(2, 6, 23, 0.1);
    }

    .ring::before {
        content: "";
        position: absolute;
        inset: 10px;
        border-radius: 999px;
        background: rgba(255, 255, 255, 0.86);
        border: 1px solid rgba(148, 163, 184, 0.25);
    }

    .time {
        position: relative;
        z-index: 1;
        text-align: center;
        font-variant-numeric: tabular-nums;
        font-size: 54px;
        font-weight: 700;
        letter-spacing: -0.04em;
        line-height: 1;
    }

    .actions {
        display: grid;
        grid-template-columns: 1.2fr 1fr 0.9fr;
        gap: 10px;
        margin: 14px 0 14px;
    }

    button {
        appearance: none;
        border: 0;
        border-radius: 14px;
        padding: 12px 14px;
        font-weight: 600;
        letter-spacing: -0.01em;
        cursor: pointer;
        transition:
            transform 0.08s ease,
            box-shadow 0.2s ease,
            background 0.2s ease;
        user-select: none;
    }

    button:active {
        transform: translateY(1px);
    }

    button:disabled {
        opacity: 0.55;
        cursor: not-allowed;
        transform: none;
    }

    .primary {
        color: white;
        background: linear-gradient(135deg, #4f46e5, #0ea5e9);
        box-shadow: 0 10px 18px rgba(79, 70, 229, 0.22);
    }

    .primary:hover:not(:disabled) {
        box-shadow: 0 14px 24px rgba(79, 70, 229, 0.26);
    }

    .secondary {
        background: rgba(15, 23, 42, 0.06);
        color: #0f172a;
    }

    .ghost {
        background: transparent;
        border: 1px solid rgba(148, 163, 184, 0.35);
        color: #0f172a;
    }

    .icon {
        width: 40px;
        height: 40px;
        display: grid;
        place-items: center;
        padding: 0;
        border-radius: 12px;
        font-size: 16px;
    }

    .panel {
        padding: 14px;
        border-radius: 16px;
        background: rgba(2, 6, 23, 0.03);
        border: 1px solid rgba(148, 163, 184, 0.2);
    }

    .panelTitle {
        font-size: 12px;
        color: #475569;
        margin: 2px 0 8px;
        font-weight: 650;
        letter-spacing: -0.01em;
    }

    .presets {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        margin-bottom: 12px;
    }

    .pill {
        padding: 9px 12px;
        border-radius: 999px;
        background: rgba(15, 23, 42, 0.06);
        color: #0f172a;
    }

    .pill.active {
        background: rgba(99, 102, 241, 0.14);
        box-shadow: inset 0 0 0 1px rgba(99, 102, 241, 0.35);
    }

    .custom {
        display: grid;
        grid-template-columns: 1fr auto;
        gap: 10px;
        align-items: end;
    }

    .field {
        display: grid;
        gap: 6px;
    }

    .label {
        font-size: 12px;
        color: #475569;
        font-weight: 600;
    }

    input {
        width: 100%;
        box-sizing: border-box;
        border-radius: 14px;
        border: 1px solid rgba(148, 163, 184, 0.4);
        background: rgba(255, 255, 255, 0.75);
        padding: 12px 12px;
        outline: none;
        font: inherit;
        font-weight: 600;
        letter-spacing: -0.01em;
        color: #0f172a;
    }

    input:focus {
        border-color: rgba(99, 102, 241, 0.55);
        box-shadow: 0 0 0 6px rgba(99, 102, 241, 0.14);
    }

    .hint {
        margin-top: 12px;
        font-size: 12px;
        color: #64748b;
        line-height: 1.35;
    }
</style>
