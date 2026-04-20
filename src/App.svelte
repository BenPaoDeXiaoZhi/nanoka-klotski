<script lang="ts" module>
  export type BlocksConfig = Record<
    number,
    Record<number, { x: number; y: number; empty?: boolean }>
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
    obj[size - 1][size - 1].empty = true;
    return obj;
  });

  let emptyPos = $derived({
    x: size - 1,
    y: size - 1,
  });

  function getBlock(pos: Pos){
    const {x,y} = pos;
    return blocks[x][y];
  }

  function setBlock(pos: Pos, movedPos: Pos){
    const {x,y} = pos;
    blocks[x][y] = movedPos;
    blocks[x][y].empty = false;
  }

  function setEmpty(pos: Pos){
    const {x,y} = pos;
    blocks[x][y].empty = true;
    emptyPos = pos;
  }

  function handleClick(pos: {
    x: number,
    y: number,
  }){
    console.log(pos, emptyPos);
    if(pos.y == emptyPos.y){
      console.log("moveX");
      if(abs(emptyPos.x - pos.x) == 1){
        const moved = getBlock(pos);
        setBlock(emptyPos, moved);
        setEmpty(pos);
      }
    }
    if(pos.x == emptyPos.x){
      console.log("moveY");
      if(abs(emptyPos.y - pos.y) == 1){
        const moved = getBlock(pos);
        setBlock(emptyPos, moved);
        setEmpty(pos);
      }
    }
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
