<script>
  import { onMount, createEventDispatcher } from "svelte";
  import { getName, getVersion } from "@tauri-apps/api/app";

  export let open = false;

  let appName = "AstraPulse";
  let version = "0.1.0";
  let license = "GPL v3.0";

  const dispatch = createEventDispatcher();

  onMount(async () => {
    try
    {
      appName = await getName();
      version = await getVersion();
    }
    catch (e)
    {
      // running in browser or permission failure
      console.warn("failed to read app metadata", e);
    }
  });

  function close()
  {
    open = false;
    dispatch("close");
  }
</script>

{#if open}
  <div
    class="about-overlay"
    role="dialog"
    aria-modal="true"
    aria-label="About dialog"
  >
    <div class="about-card">
      <header class="about-header">
        <div>
          <div class="about-title">{appName}</div>
          <div class="about-sub">Version {version}</div>
        </div>
        <button class="icon" aria-label="Close" on:click={close}>✕</button>
      </header>

      <div class="about-body">
        <p>
          AstraPulse is a small, focused Pomodoro-style timer to help you stay
          productive. It is built using a Svelte frontend inside a Tauri wrapper
          for a native desktop experience.
        </p>

        <p>
          Website: <a
            href="https://sandboxscience.vercel.app/"
            target="_blank"
            rel="noopener">sandboxscience.vercel.app</a
          >
        </p>

        <p class="muted">© SandBox Science — License: {license}</p>
      </div>

      <footer class="about-footer">
        <button class="secondary" on:click={close}>Close</button>
      </footer>
    </div>
  </div>
{/if}

<style>
  .about-overlay {
    position: fixed;
    inset: 0;
    display: grid;
    place-items: center;
    padding: 24px;
    background: rgba(2, 6, 23, 0.55);
    backdrop-filter: blur(10px);
    z-index: 60;
  }

  .about-card {
    width: min(560px, 100%);
    background: rgba(255, 255, 255, 0.78);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(148, 163, 184, 0.22);
    border-radius: 20px;
    padding: 18px;
    box-shadow:
      0 26px 70px rgba(2, 6, 23, 0.28),
      0 2px 10px rgba(2, 6, 23, 0.1);
    color: #0f172a;
    overflow: hidden;
  }

  .about-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 14px;
    padding-bottom: 12px;
    border-bottom: 1px solid rgba(148, 163, 184, 0.22);
    margin-bottom: 12px;
  }

  .about-title {
    display: flex;
    align-items: center;
    gap: 10px;
    font-weight: 750;
    font-size: 16px;
    letter-spacing: -0.02em;
    line-height: 1.2;
  }

  /* Small brand dot (matches your main UI style) */
  .about-title::before {
    content: "";
    width: 10px;
    height: 10px;
    border-radius: 999px;
    background: linear-gradient(135deg, #6366f1, #0ea5e9);
    box-shadow: 0 0 0 6px rgba(99, 102, 241, 0.12);
    flex: 0 0 auto;
  }

  .about-sub {
    font-size: 12px;
    color: #475569;
    margin-top: 4px;
    letter-spacing: -0.01em;
  }

  .about-body {
    font-size: 13px;
    line-height: 1.55;
    color: #0f172a;
  }

  .about-body p {
    margin: 10px 0;
  }

  .about-body a {
    color: #1d4ed8;
    text-decoration: none;
    font-weight: 650;
  }

  .about-body a:hover {
    text-decoration: underline;
  }

  .muted {
    color: #64748b;
    font-size: 12px;
    margin-top: 12px;
  }

  .about-footer {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    gap: 10px;
    padding-top: 14px;
    margin-top: 12px;
    border-top: 1px solid rgba(148, 163, 184, 0.22);
  }

  .icon {
    width: 40px;
    height: 40px;
    display: grid;
    place-items: center;
    border-radius: 12px;
    background: rgba(15, 23, 42, 0.06);
    border: 1px solid rgba(148, 163, 184, 0.22);
    color: #0f172a;
    cursor: pointer;
    transition:
      transform 0.08s ease,
      background 0.2s ease,
      box-shadow 0.2s ease;
    user-select: none;
  }

  .icon:hover {
    background: rgba(15, 23, 42, 0.08);
    box-shadow: 0 10px 18px rgba(2, 6, 23, 0.08);
  }

  .icon:active {
    transform: translateY(1px);
  }

  .secondary {
    appearance: none;
    border: 1px solid rgba(148, 163, 184, 0.28);
    background: rgba(15, 23, 42, 0.06);
    color: #0f172a;
    padding: 10px 14px;
    border-radius: 14px;
    font-weight: 650;
    letter-spacing: -0.01em;
    cursor: pointer;
    transition:
      transform 0.08s ease,
      box-shadow 0.2s ease,
      background 0.2s ease;
  }

  .secondary:hover {
    background: rgba(15, 23, 42, 0.08);
    box-shadow: 0 10px 18px rgba(2, 6, 23, 0.08);
  }

  .secondary:active {
    transform: translateY(1px);
  }

  @media (prefers-reduced-motion: reduce) {
    .icon,
    .secondary {
      transition: none;
    }
  }
</style>
