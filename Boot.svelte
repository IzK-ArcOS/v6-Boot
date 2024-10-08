<script lang="ts">
  import GlowingLogo from "$lib/Components/GlowingLogo.svelte";
  import Spinner from "$lib/Components/Spinner.svelte";
  import { SafeMode } from "$state/Desktop/ts/store";
  import { Log } from "$ts/console";
  import { ArcOSVersion } from "$ts/env";
  import { BugReportIcon } from "$ts/images/general";
  import { ARCOS_BUILD, ARCOS_MODE } from "$ts/metadata";
  import { isDesktop } from "$ts/metadata/desktop";
  import { getAuthcode } from "$ts/server/authcode";
  import { getAllServers, getServer } from "$ts/server/multi";
  import { testConnection } from "$ts/server/test";
  import { PrimaryState, StateHandler } from "$ts/states";
  import { sleep } from "$ts/util";
  import { onMount } from "svelte";
  import "./css/boot.css";

  export let handler: StateHandler;

  let status = "";
  let bootClass = "";
  let targetState = "login";
  let progress = false;
  let running = false;
  let connectFailed = false;
  let isSafeMode = false;

  const { current: State } = handler;

  onMount(async () => {
    document.addEventListener("keydown", safeMode);
    document.addEventListener("keydown", arcTermShortcut);

    if (isDesktop()) {
      await sleep(500);

      bootClass = "fadein";

      return startBooting();
    }

    status = "Press a key or click to start";

    document.addEventListener("click", startBooting, { once: true });
    document.addEventListener("keydown", startBooting, { once: true });

    await sleep(500);

    bootClass = "fadein";
  });

  async function startBooting() {
    if (running) return;
    running = true;
    status = "&nbsp;";
    progress = true;

    if (targetState == "serverselect") return await redirect();

    if (!(await checkServer()) && !getAllServers().length) status = "Welcome to ArcOS";

    if (!connectFailed) await redirect();
  }

  async function safeMode(e: KeyboardEvent) {
    if (!e.key || $State.key !== "boot") return;

    const key = e.key.toLowerCase();

    if (key != " " || !e.shiftKey) return;

    status = "Entering Safe Mode...";
    SafeMode.set(true);
    isSafeMode = true;
  }

  function arcTermShortcut(e: KeyboardEvent) {
    if (!e.key || isSafeMode || $State.key !== "boot") return;

    const key = e.key.toLowerCase();

    if (key == "f8" || targetState == "serverselect") {
      status = "Waiting for Server Select...";
      return (targetState = "serverselect");
    }

    if (!e.altKey || key != "a") return;

    targetState = "arcterm";
    status = "Loading ArcTerm";
  }

  async function checkServer() {
    const serverHost = getServer();
    const authCode = getAuthcode(serverHost);

    if (!serverHost) return;

    if (authCode) status = "Connecting Securely";

    const connected = await testConnection(serverHost, authCode);

    if (!connected) {
      // manualCrash("Boot", `Connecting to server failed. This is a temporary error.`);
      connectFailed = true;

      targetState = "boot";

      return /* targetState == "serverselect" ? false : true; */ false;
    }

    return connected;
  }

  async function redirect() {
    Log("Boot", "Redirecting");

    await sleep(2000);
    bootClass = "fadeout";
    await sleep(750);
    handler.navigate(targetState);
    //Busy.set(false);
  }

  async function changeServer() {
    targetState = "serverselect";
    await sleep(2000);
    PrimaryState.navigate("serverselect");
  }
</script>

<div class="state-boot fullscreen center-flex {bootClass}">
  {#if connectFailed}
    <div class="connection-failed">
      <img src={BugReportIcon} alt="" />
      <h1>Connection failed!</h1>
      <p>
        ArcOS could not connect to <b>{getServer()}</b>!<br />
        Please check your internet connection.
      </p>
      <button on:click={changeServer}>Change Server</button>
      <button on:click={() => location.reload()}>Restart</button>
    </div>
  {:else}
    <div class="content">
      <GlowingLogo />
      <div class="bottom">
        <Spinner height={30} stopped={!progress} />
        <p class="status">{@html status}</p>
      </div>
    </div>
  {/if}
  <div class="version">v{ArcOSVersion}-{ARCOS_MODE} ({ARCOS_BUILD})</div>
  <div class="f8">
    {targetState == "serverselect" ? "Loading Server Selector..." : "Press F8 to select server"}
  </div>
</div>
