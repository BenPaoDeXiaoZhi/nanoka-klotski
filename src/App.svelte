<script lang="ts" module>
  export type BlocksConfig = Record<number, Pos>;
  export type Pos = { x: number; y: number };

  const { abs } = Math;
</script>

<script lang="ts">
  import BigBorder from "./BigBorder.svelte";
  import Header from "./Header.svelte";
  import Block from "./Block.svelte";

  let size = $state(3);
  let showNum = $state(false);
  let blocks = $derived.by(() => {
    const obj: BlocksConfig = {};
    for (let x = 0; x < size; x++) {
      for (let y = 0; y < size; y++) {
        obj[y * size + x] = { x, y };
      }
    }
    return obj;
  });

  let emptyPos: Pos = $derived(blocks[size * size - 1]);

  function getBlock(id: number) {
    return blocks[id];
  }

  function handleClick(id: number) {
    const blockPos = getBlock(id);
    const { x, y } = blockPos;
    if (emptyPos.x == x) {
      if (abs(emptyPos.y - y) == 1) {
        blocks[size * size - 1] = blockPos;
        blocks[id] = emptyPos;
        blocks = Object.assign({}, blocks); // make derived update
      }
    }
    console.log(blocks);
  }
</script>

<Header />
<BigBorder {size} --margin="{0.75 / size}em">
  {#each { length: size * size }, id}
    <Block
      {id}
      {size}
      {blocks}
      {showNum}
      {handleClick}
    />
  {/each}
</BigBorder>

<label>大小<input bind:value={size} /></label>
<label>显示数字<input type="checkbox" bind:checked={showNum} /></label>