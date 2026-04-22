<script lang="ts" module>
  export type BlocksConfig = Pos[];
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
    const obj: BlocksConfig = [];
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

  function getId(pos: Pos){
    const {x,y} = pos;
    return blocks.findIndex((b)=>b.x==x && b.y==y) ?? -1;
  }

  function nextTo(pos: Pos){
    const { x, y } = pos;
    return abs(emptyPos.y - y) == 1 || abs(emptyPos.x - x) == 1;
  }

  function sameLine(pos: Pos){
    const { x, y } = pos;
    return emptyPos.x == x || emptyPos.y == y;
  }

  function move(id: number){
    const blockPos = getBlock(id);
    if(!sameLine(blockPos)){
      console.error("err pos");
      return;
    }
    if(nextTo(blockPos)){
      blocks[size * size - 1] = blockPos;
      blocks[id] = emptyPos;
      return;
    }
    const {x,y}=blockPos;
    const delta = emptyPos.y - y + emptyPos.x - x
    if(x==emptyPos.x){
      const next = y + delta / abs(delta);
      const nextId = getId({x,y:next});
      move(nextId);
      move(id);
    }
    if(y==emptyPos.y){
      const next = x + delta / abs(delta);
      const nextId = getId({x:next,y});
      move(nextId);
      move(id);
    }
  }

  function handleClick(id: number) {
    const blockPos = getBlock(id);
    if (sameLine(blockPos)) {
      move(id);
      blocks = Object.assign({}, blocks); // make derived update
    }
    console.log(blocks);
  }
</script>

<Header />
<BigBorder {size}>
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

<style>
:root {
  font-size: calc(min(90vw, 90vh) / 30);
}
</style>