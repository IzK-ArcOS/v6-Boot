<script lang="ts">
  import "./css/boot.css";
  import { Logo } from "$ts/branding";
  import { StateHandler } from "$ts/states";
  import { onMount } from "svelte";
  import Indeterminate from "$lib/Components/Progress/Indeterminate.svelte";

  export let handler: StateHandler;

  onMount(() => {
    handler.navigate("login"); // !!! dummy, won't do anything yet
  });
</script>

<div class="state-boot fullscreen center-flex">
  <div class="content">
    <img src={Logo()} alt="" class="logo" />
    <Indeterminate width={150} />
  </div>
</div>
