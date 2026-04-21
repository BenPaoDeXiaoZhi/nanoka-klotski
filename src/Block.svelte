<script lang="ts">
  import type { BlocksConfig } from "./App.svelte";

  type Props = {
    id: number;
    size: number;
    blocks: BlocksConfig;
    showNum: boolean;
    handleClick: (id: number) => any;
  };
  const { id, size, blocks, showNum, handleClick }: Props = $props();
  let posX = $derived(blocks[id].x);
  let posY = $derived(blocks[id].y);
  let x = $derived(id % size);
  let y = $derived((id - (id % size)) / size);
  let empty = $derived(id == size * size - 1);
</script>

<div class="hole"
  style:width="{30 / size}em"
  style:top="{y * 30 / size}em"
  style:left="{x * 30 / size}em"
></div>

{#if !empty}
<button
  class="block"
  onclick={() => {
    if (empty) return;
    handleClick(id);
  }}
  style:width="{30 / size}em"
  style:top="{posY * 30 / size}em"
  style:left="{posX * 30 / size}em"
>
  <div 
    class="container"
    style:box-shadow={empty ? "none" : "2px 2px 0.3em rgb(181, 181, 181)"}
  >
    <div
      class="img"
      style:top="{(-y * 30) / size}em"
      style:left="{(-x * 30) / size}em"
    >
      <img src="./nanoka.png" alt="nnk" />
    </div>
    {#if showNum}
      <b>{id + 1}</b>
    {/if}
  </div>
</button>
{/if}

<style>
  button.block {
    position: absolute;
    aspect-ratio: 1;
    border: none;
    font-size: 1em;
    transition: 0.1s;
    background-color: transparent;
  }

  .container {
    margin: var(--margin);
    aspect-ratio: 1;
    width: calc(100% - var(--margin) * 2);
    border-radius: calc(var(--margin) * 2);
    overflow: hidden;
    position: relative;
  }

  div.hole {
    position: absolute;
    width: 100%;
    aspect-ratio: 1;
    border-radius: calc(var(--margin) * 2);
    background-color: #b1b1b1;
    box-shadow: inset 1px 1px 0.5em 0px rgb(97, 97, 97);
    z-index: 2;
  }

  div.img {
    width: 100%;
    aspect-ratio: 1;
    position: relative;
  }

  img {
    width: 30em;
    aspect-ratio: 1;
    z-index: 3;
  }

  b {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 5;
    font-size: calc(var(--margin) * 12);
    color: white;
  }
</style>
