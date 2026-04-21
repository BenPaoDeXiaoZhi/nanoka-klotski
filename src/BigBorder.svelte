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
  div .out {
    width: 31em;
    aspect-ratio: 1;
    border-radius: 0.5em;
    background-color: #e0e2e4;
    padding: 0.5em;
  }

  div .in {
    width: 30em;
    aspect-ratio: 1;
    position: relative;
  }
</style>
