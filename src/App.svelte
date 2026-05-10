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
    func: (i: number) => any,
    end = () => {},
  ) {
    let i = 0;

    if (delay == 0) {
      while (1) {
        func(i);

        if (i++ == times) {
          end();

          return;
        }
      }
    } else {
      const id = setInterval(() => {
        func(i);

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
  import Timer from "./Timer.svelte";
  import Button from "$lib/components/ui/button/button.svelte";
  import * as AlertDialog from "$lib/components/ui/alert-dialog";
  import Label from "$lib/components/ui/label/label.svelte";
  import Input from "$lib/components/ui/input/input.svelte";
  import Switch from "$lib/components/ui/switch/switch.svelte";

  let size = $state(3);
  let showNum = $state(false);
  let showWinDialog = $state(false);
  let showShuffle = $state(false);
  let shuffling = $derived(false);
  let start = $derived(false);
  let timeStr = $state("");

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
    start = false;
    timeStr = "00:00:00.000";

    const times = size * size * randInt(1, 10);

    console.log(`将进行${times}次打乱`);

    const delay = showShuffle ? 10 : 0;

    interval(
      times,
      delay,
      (i) => {
        const { x, y } = blocks[size * size - 1];

        // const moveDir = randInt(0, 1);
        const moveDir = i % 2;

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
      if (!start) start = true;
      move(id);
      blocks = Array.from(blocks); // make derived update
    }

    if (checkIsWin()) {
      start = false;
      showWinDialog = true;
    }
  }

  function checkIsWin() {
    return !blocks.find((pos, i) => pos.y * size + pos.x !== i);
  }
</script>

<Header />

<div class="container flex flex-wrap select-none m-1 max-w-max">
  <BigBorder {size}>
    {#each { length: size * size }, id}
      <Block bind:start {id} {size} {blocks} {showNum} {handleClick} />
    {/each}
  </BigBorder>

  <div class="flex flex-col m-1 gap-1 w-40">
    <Label class="text-nowrap">
      大小
      <Input type="number" bind:value={size} class="w-max" />
    </Label>

    <Label>
      显示数字
      <Switch bind:checked={showNum} class="ml-auto mr-0" />
    </Label>

    <Label>
      显示打乱过程
      <Switch bind:checked={showShuffle} class="ml-auto mr-0" />
    </Label>

    <Button
      variant="outline"
      class="bg-gray-200 hover:bg-gray-300 border-gray-400 "
      onclick={() => shuffle()}
      disabled={shuffling}
    >
      打乱
    </Button>
  </div>

  <Timer bind:start bind:timeStr time={0} />

  <AlertDialog.Root bind:open={showWinDialog}>
    <AlertDialog.Content>
      <AlertDialog.Header class="m-auto">胜利!</AlertDialog.Header>
      <AlertDialog.Description>
        梅露露酱仅用{timeStr}就结束了比赛!<br />大魔女十分欣慰~
      </AlertDialog.Description>
      <AlertDialog.Footer>
        <AlertDialog.Action
          onclick={() => (showWinDialog = false)}
          class="m-auto w-sm"
        >
          继续
        </AlertDialog.Action>
      </AlertDialog.Footer>
    </AlertDialog.Content>
  </AlertDialog.Root>
</div>

<style lang="postcss">
  @reference "tailwindcss";
  :global(#app) {
    font-size: calc(min(3.14vw, 2.4vh));
  }
</style>
