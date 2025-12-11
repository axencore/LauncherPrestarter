<script lang="ts">
  import { onDestroy } from "svelte";

  export let percentage: number = 0;

  onDestroy(() => (percentage = 0));
</script>

<div class={"progress-bar " + ($$restProps.class || "")}>
  <div style="width: {percentage}%; height: 100%;"></div>
</div>

<style lang="scss">
  .progress-bar {
    width: 100%;
    height: 22px;
    border-radius: 80px;
    border: 3px solid #c4ac7c;
    background: $base;
    backdrop-filter: blur(4px);
    overflow: hidden;
    position: relative;

    > div {
      background: $progressbar;
      height: 100%;
      border-radius: 80px;
      box-shadow: 0 0 18px rgba(196, 172, 124, 0.45);
      transition: width 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
    }

    &.errored {
      background: rgba($error, 0.15);
      border-color: $error;

      > div {
        background: $error;
        box-shadow: none;
      }
    }
  }
</style>
