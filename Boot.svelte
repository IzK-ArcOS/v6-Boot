<script lang="ts">
  import { Logo } from "$ts/branding";
  import { StateHandler } from "$ts/states";
  import { onMount } from "svelte";
  import Bar from "../../lib/Components/Progress/Bar.svelte";

  export let handler: StateHandler;

  onMount(() => {
    handler.navigate("login"); // !!! dummy, won't do anything yet
  });
</script>

<div class="state-boot fullscreen center-flex">
  <div class="content">
    <img src={Logo()} alt="" class="logo" />
    <Bar indeterminate />
  </div>
</div>
