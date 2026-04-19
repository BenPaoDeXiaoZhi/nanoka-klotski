<script lang="ts">
  import type { Snippet } from "svelte";

  type Props = {
    children: Snippet;
    size: number;
  };
  const { children, size }: Props = $props();
</script>

<div
  style:grid-template-columns="repeat({size}, auto)"
  style:grid-template-rows="repeat({size}, auto)"
>
  {@render children()}
</div>

<style>
  div {
    width: 30em;
    display: grid;
    aspect-ratio: 1;
    border-radius: 0.5em;
    background-color: #e0e2e4;
    padding: 0.25em;
  }
</style>
