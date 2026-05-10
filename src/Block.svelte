<script lang="ts">
  import type { BlocksConfig } from "./App.svelte";

  type Props = {
    id: number;
    size: number;
    blocks: BlocksConfig;
    showNum: boolean;
    handleClick: (id: number) => any;
    start: boolean;
  };
  let {
    id,
    size,
    blocks,
    showNum,
    handleClick,
    start = $bindable(),
  }: Props = $props();
  let posX = $derived(blocks[id].x);
  let posY = $derived(blocks[id].y);
  let x = $derived(id % size);
  let y = $derived((id - (id % size)) / size);
  let empty = $derived(id == size * size - 1);
</script>

<div
  class="aspect-square absolute"
  style:width="{30 / size}em"
  style:top="{(y * 30) / size}em"
  style:left="{(x * 30) / size}em"
>
  <div
    class="relative aspect-square w-[90%] overflow-hidden m-[5%]"
    style:border-radius="{3 / size}em"
  >
    <div class="bg-gray-300 z-2 shadow-inset-md size-full"></div>
  </div>
</div>

{#if !empty}
  <button
    class="absolute aspect-square border-0 text-[1em] duration-100 z-3 m-0"
    onclick={() => {
      if (empty) return;
      handleClick(id);
    }}
    style:width="{30 / size}em"
    style:top="{(posY * 30) / size}em"
    style:left="{(posX * 30) / size}em"
    style:transition-property={start ? "top, left" : "none"}
  >
    <div
      class="relative aspect-square w-[90%] overflow-hidden m-[5%] shadow-md"
      style:border-radius="{3 / size}em"
      style:background="url('./nanoka.png')"
      style:background-position="{(-x * 30) / size}em {(-y * 30) / size}em"
      style:background-size="30em"
    >
      {#if showNum}
        <b
          class="absolute top-[-.25em] left-0 text-white z-5 font-sans"
          style:font-size="{12 / size}em">{id + 1}</b
        >
      {/if}
    </div>
  </button>
{/if}
