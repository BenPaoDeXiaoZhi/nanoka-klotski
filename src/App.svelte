<script lang="ts" module>
  export type BlocksConfig = Record<
    number,
    Record<number, { x: number; y: number; empty?: boolean }>
  >;
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

  function handleClick(args: {
    x: number,
    y: number,
  }){
    const {x, y} = args;
    console.log(args, emptyPos);
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
