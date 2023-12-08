<script lang="ts">
  import Indeterminate from "$lib/Components/Progress/Indeterminate.svelte";
  import { Logo } from "$ts/branding";
  import { Log } from "$ts/console";
  import { isDesktop } from "$ts/metadata/desktop";
  import { getAuthcode } from "$ts/server/authcode";
  import { getAllServers, getServer } from "$ts/server/multi";
  import { testConnection } from "$ts/server/test";
  import { StateHandler } from "$ts/states";
  import { sleep } from "$ts/util";
  import { onMount } from "svelte";
  import "./css/boot.css";

  export let handler: StateHandler;

  let status = "";
  let bootClass = "";
  let targetState = "arcterm";
  let progress = false;
  let running = false;

  onMount(async () => {
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
    //Busy.set(true);

    if (targetState == "serverselect") return await redirect();

    if (!(await checkServer()) && !getAllServers()) status = "Preparing ArcOS";

    await redirect();
  }

  async function checkServer() {
    const serverHost = getServer();
    const authCode = getAuthcode(serverHost);

    if (!serverHost) return;

    if (authCode) status = "Connecting Securely";

    const connected = await testConnection(serverHost, authCode);

    if (!connected) {
      bootClass = "fadeout";

      return targetState == "serverselect" ? false : true;
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
</script>

<div class="state-boot fullscreen center-flex {bootClass}">
  <div class="content">
    <img src={Logo()} alt="" class="logo" />
    <Indeterminate width={150} active={progress} />
    <p class="status">{@html status}</p>
  </div>
</div>
