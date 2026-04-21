<script lang="ts" module>
  export type BlocksConfig = Record<
    number,
    Record<number, Pos>
  >;
  export type Pos = { x: number; y: number; };

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
      obj[x] = {};
      for (let y = 0; y < size; y++) {
        obj[x][y] = { x, y };
      }
    }
    return obj;
  });

  let emptyPos: Pos = $derived(blocks[size - 1][size - 1]);

  function getBlock(picPos: Pos){
    const {x,y} = picPos;
    return blocks[x][y];
  }

  function handleClick(picPos: Pos){
    const blockPos = getBlock(picPos);
    const {x,y} = blockPos;
    if(emptyPos.x == x){
      if(abs(emptyPos.y - y)==1){
        blocks[size - 1][size - 1] = blockPos;
        blocks[x][y] = emptyPos;
        blocks = Object.assign({},blocks); // make derived update
      }
    }
    console.log(blocks);
  }
</script>

<Header />
<BigBorder {size}>
  {#each { length: size }, y}
    {#each { length: size }, x}
      <Block {x} {y} {size} {blocks} {showNum} {handleClick} --margin="{0.75 / size}em" />
    {/each}
  {/each}
</BigBorder>

<label>大小<input bind:value={size} /></label>
<label>显示数字<input type="checkbox" bind:checked={showNum} /></label>
