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
  class="out"
  style:width="{30 / size}em"
  style:top="{(y * 30) / size}em"
  style:left="{(x * 30) / size}em"
>
  <div class="container">
    <div class="hole"></div>
  </div>
</div>

<button
  class="block"
  onclick={() => {
    if (empty) return;
    if (!start) start = true;
    handleClick(id);
  }}
  style:width="{30 / size}em"
  style:top="{(posY * 30) / size}em"
  style:left="{(posX * 30) / size}em"
>
  <div
    class="container"
    style:box-shadow={empty ? "none" : "2px 2px 0.3em rgb(181, 181, 181)"}
  >
    {#if !empty}
      <div
        class="img"
        style:top="{(-y * 30) / size}em"
        style:left="{(-x * 30) / size}em"
      >
        <img src="./nanoka.png" alt="nnk" />
      </div>
      {#if showNum}
        <b style:font-size="{12 / size}em">{id + 1}</b>
      {/if}
    {/if}
  </div>
</button>

<style>
  div.out {
    position: absolute;
    padding: 0em;
    aspect-ratio: 1;
  }

  div.hole {
    position: absolute;
    width: 100%;
    aspect-ratio: 1;
    border-radius: 10%;
    background-color: #b1b1b1;
    box-shadow: inset 1px 1px 0.5em 0px rgb(97, 97, 97);
    z-index: 2;
  }

  button.block {
    position: absolute;
    aspect-ratio: 1;
    border: none;
    font-size: 1em;
    transition: 0.1s;
    background-color: transparent;
    z-index: 3;
    padding: 0em;
  }

  .container {
    aspect-ratio: 1;
    width: 90%;
    border-radius: 10%;
    overflow: hidden;
    position: relative;
    margin: 5%;
  }

  div.img {
    width: 100%;
    aspect-ratio: 1;
    position: absolute;
  }

  img {
    width: 30em;
    aspect-ratio: 1;
    user-select: none;
    -webkit-user-drag: none;
  }

  b {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 5;
    color: white;
  }
</style>
