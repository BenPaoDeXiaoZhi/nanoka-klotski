<script lang="ts">
  import type { BlocksConfig } from "./App.svelte";

  type Props = {
    x: number;
    y: number;
    size: number;
    blocks: BlocksConfig;
  };
  const { x, y, size, blocks }: Props = $props();
  let movedX = $derived(blocks[x][y].x);
  let movedY = $derived(blocks[x][y].y);
  let empty = $derived(blocks[x][y].empty);
</script>

<div
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
</div>

<style>
  div.block {
    width: calc(100% - 0.5em);
    aspect-ratio: 1;
    margin: 0.25em;
    border-radius: 0.5em;
    overflow: hidden;
  }

  div.hole {
    width: 100%;
    aspect-ratio: 1;
    border-radius: 0.5em;
    box-shadow: inset 1px 1px 0.5em 0px rgb(97, 97, 97);
  }

  div.img {
    width: 100%;
    aspect-ratio: 1;
    position: relative;
  }

  img {
    height: 30em;
  }
</style>
