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
  let empty = $derived(id == size * size);
</script>

<button
  class="block"
  onclick={() => {
    if (empty) return;
    handleClick(id);
  }}
  style:grid-row={posY}
  style:grid-column={posX}
  style:background-color={empty ? "#b1b1b1" : "white"}
  style:box-shadow={empty ? "none" : "2px 2px 0.3em rgb(181, 181, 181)"}
>
  {#if empty}
    <div class="hole"></div>
  {:else}
    <div
      class="img"
      style:top="{(-y * 30) / size}em"
      style:left="{(-x * 30) / size}em"
    >
      <img src="./nanoka.png" alt="nnk" />
    </div>
    {#if showNum}
      <b>{id}</b>
    {/if}
  {/if}
</button>

<style>
  button.block {
    position: relative;
    width: calc(100% - var(--margin) * 2);
    aspect-ratio: 1;
    margin: var(--margin);
    border-radius: calc(var(--margin) * 2);
    overflow: hidden;
    padding: 0px;
    border: none;
    font-size: 1em;
  }

  div.hole {
    width: 100%;
    aspect-ratio: 1;
    border-radius: calc(var(--margin) * 2);
    box-shadow: inset 1px 1px 0.5em 0px rgb(97, 97, 97);
  }

  div.img {
    width: 100%;
    aspect-ratio: 1;
    position: relative;
  }

  img {
    width: 30em;
    aspect-ratio: 1;
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
