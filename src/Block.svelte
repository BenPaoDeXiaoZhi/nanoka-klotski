<script lang="ts">
  import type { BlocksConfig } from "./App.svelte";

  type Props = {
    x: number;
    y: number;
    size: number;
    blocks: BlocksConfig;
    showNum: boolean;
  };
  const { x, y, size, blocks, showNum }: Props = $props();
  let movedX = $derived(blocks[x][y].x);
  let movedY = $derived(blocks[x][y].y);
  let empty = $derived(blocks[x][y].empty);
</script>

<button
  class="block"
  style:grid-row={y + 1}
  style:grid-column={x + 1}
  style:background-color={empty ? "#b1b1b1" : "white"}
  style:box-shadow={empty ? "none" : "2px 2px 0.3em rgb(181, 181, 181)"}
>
  {#if empty}
    <div class="hole"></div>
  {:else}
    <div
      class="img"
      style:top="{(-movedY * 30) / size}em"
      style:left="{(-movedX * 30) / size}em"
    >
      <img src="./nanoka.png" alt="nnk" />
    </div>
  {/if}
  {#if showNum}
    <b>{y * size + x + 1}</b>
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
    color: white;
  }
</style>
