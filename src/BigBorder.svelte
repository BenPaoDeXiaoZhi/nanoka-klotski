<script lang="ts">
  import type { Snippet } from "svelte";

  type Props = {
    children: Snippet;
    size: number;
  };
  const { children, size }: Props = $props();
</script>

<div
  class="size-[31em] border-0 rounded-md bg-gray-200 p-[0.5em] relative top-0 left-0 shrink-0"
>
  <div class="relative">
    {@render children()}
  </div>
</div>
