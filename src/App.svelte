<script lang="ts" module>
  export type BlocksConfig = Pos[];
  export type Pos = { x: number; y: number };

  const { abs } = Math;

  function randInt(a = 0, b: number) {
    return Math.floor(a + Math.random() * (b - a + 1));
  }

  function interval(
    times: number,
    delay: number,
    func: Function,
    end = () => {},
  ) {
    let i = 0;
    if (delay == 0) {
      while (1) {
        func();
        if (i++ == times) {
          end();
          return;
        }
      }
    } else {
      const id = setInterval(() => {
        func();
        if (i++ == times) {
          clearInterval(id);
          end();
        }
      }, delay);
    }
  }
</script>

<script lang="ts">
  import BigBorder from "./BigBorder.svelte";
  import Header from "./Header.svelte";
  import Block from "./Block.svelte";

  let size = $state(3);
  let showNum = $state(false);
  let showShuffle = $state(false);
  let shuffling = $derived(false);
  let blocks = $derived.by(() => {
    const obj: BlocksConfig = [];
    for (let x = 0; x < size; x++) {
      for (let y = 0; y < size; y++) {
        obj[y * size + x] = { x, y };
      }
    }
    return obj;
  });

  function getBlock(id: number) {
    return blocks[id];
  }

  function getId(pos: Pos) {
    const { x, y } = pos;
    return blocks.findIndex((b) => b.x == x && b.y == y) ?? -1;
  }

  function sameLine(pos: Pos) {
    const { x, y } = pos;
    return blocks[size * size - 1].x == x || blocks[size * size - 1].y == y;
  }

  function fn(a: number, b: number, l: number) {
    if (b == a) {
      return a;
    }
    return a + ((b - a) / abs(b - a)) * l;
  }

  function move(id: number) {
    const blockPos = getBlock(id);
    if (!sameLine(blockPos)) {
      console.error("err pos");
      return;
    }
    const { x: srcX, y: srcY } = blockPos;
    const { x: dstX, y: dstY } = blocks[size * size - 1];
    let n = 1;
    while (1) {
      const prevX = fn(dstX, srcX, n - 1);
      const prevY = fn(dstY, srcY, n - 1);
      const currentX = fn(dstX, srcX, n);
      const currentY = fn(dstY, srcY, n++);
      const currentId = getId({ x: currentX, y: currentY });
      blocks[currentId] = { x: prevX, y: prevY };
      if (currentY == srcY && currentX == srcX) {
        blocks[size * size - 1] = { x: srcX, y: srcY };
        break;
      }
    }
    blocks = Array.from(blocks);
  }

  function shuffle() {
    shuffling = true;
    const times = size * size * randInt(1, 10);
    console.log(`将进行${times}次打乱`);
    const delay = showShuffle ? 10 : 0;
    interval(
      times,
      delay,
      () => {
        const { x, y } = blocks[size * size - 1];
        const moveDir = randInt(0, 1);
        let movePos: Pos = { x, y };
        while (1) {
          if (moveDir == 0) {
            movePos = { x: randInt(0, size - 1), y };
          } else {
            movePos = { x, y: randInt(0, size - 1) };
          }
          if (!(movePos.x == x && movePos.y == y)) {
            break;
          }
        }
        const moveId = getId(movePos);
        move(moveId);
      },
      () => {
        resetEmpty();
        shuffling = false;
      },
    );
  }

  function resetEmpty() {
    if (blocks[size * size - 1].x !== size - 1) {
      const movePos = { x: size - 1, y: blocks[size * size - 1].y };
      const moveId = getId(movePos);
      move(moveId);
    }
    if (blocks[size * size - 1].y !== size - 1) {
      const movePos = { x: blocks[size * size - 1].x, y: size - 1 };
      const moveId = getId(movePos);
      move(moveId);
    }
  }

  function handleClick(id: number) {
    const blockPos = getBlock(id);
    if (sameLine(blockPos)) {
      move(id);
      blocks = Array.from(blocks); // make derived update
    }
    if(verify){
      alert("已复原");
    }
  }

  function verify(){
    return !blocks.find(
      (pos, i) => pos.y * size + pos.x == i;
    );
  }
</script>

<Header />
<div class="wrapper">
  <BigBorder {size}>
    {#each { length: size * size }, id}
      <Block {id} {size} {blocks} {showNum} {handleClick} />
    {/each}
  </BigBorder>
  <div class="input">
    <label>大小<input bind:value={size} /></label>
    <label>显示数字<input type="checkbox" bind:checked={showNum} /></label>
    <label
      >显示打乱过程<input type="checkbox" bind:checked={showShuffle} /></label
    >
    <button onclick={() => shuffle()} disabled={shuffling}>打乱</button>
  </div>
</div>

<style>
  :root {
    font-size: calc(min(3.25vw, 2.5vh));
  }

  .wrapper {
    display: flex;
    flex-wrap: wrap;
    user-select: none;
  }

  .input {
    display: flex;
    flex-direction: column;
  }

  label {
    font-size: 1em;
  }

  input {
    width: 2em;
    font-size: 1em;
  }
</style>
