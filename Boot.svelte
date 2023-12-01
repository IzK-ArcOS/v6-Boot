<script lang="ts">
  import "./css/boot.css";
  import { Logo } from "$ts/branding";
  import { StateHandler } from "$ts/states";
  import { onMount } from "svelte";
  import Indeterminate from "$lib/Components/Progress/Indeterminate.svelte";
  import { sleep } from "$ts/util";
  import { Log } from "$ts/console";
  import { getAllServers, getServer } from "$ts/server/multi";
  import { getAuthcode } from "$ts/server/authcode";
  import { testConnection } from "$ts/server/test";

  export let handler: StateHandler;

  let status = "";
  let bootClass = "";
  let targetState = "login";
  let progress = false;
  let running = false;

  onMount(async () => {
    status = "Press a key or click to start";

    document.addEventListener("click", startBooting, { once: true });
    document.addEventListener("keydown", startBooting, { once: true });
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
    <Indeterminate width={150} />
    <p class="status">{@html status}</p>
  </div>
</div>
