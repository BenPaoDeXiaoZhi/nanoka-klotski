<script lang="ts">
  import type { Snippet } from "svelte";

  type Props = {
    children: Snippet;
    size: number;
  };
  const { children, size }: Props = $props();
</script>

<div class="out">
  <div class="in">
    {@render children()}
  </div>
</div>

<style>
  div.out {
    width: 31em;
    height: 31em;
    border-radius: 0.5em;
    background-color: #e0e2e4;
  }

  div.in {
    width: 30em;
    aspect-ratio: 1;
    position: relative;
    top: 0.5em;
    left: 0.5em;
    z-index: 1;
    background-color: #e0e2e4;
  }
</style>
