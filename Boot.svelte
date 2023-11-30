<script lang="ts">
  import { Logo } from "$ts/branding";
  import Bar from "../../lib/Components/Progress/Bar.svelte";
</script>

<div class="state-boot fullscreen center-flex">
  <div class="content">
    <img src={Logo()} alt="" class="logo" />
    <Bar indeterminate />
  </div>
</div>
