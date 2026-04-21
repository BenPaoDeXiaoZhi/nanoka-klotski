<script lang="ts">
  import type { Snippet } from "svelte";

  type Props = {
    children: Snippet;
    size: number;
  };
  const { children, size }: Props = $props();
</script>

<div>
  {@render children()}
</div>

<style>
  div.out {
    width: 30em;
    height: 30em;
    border-radius: 0.5em;
    background-color: #e0e2e4;
    padding: 0.5em
  }
</style>
