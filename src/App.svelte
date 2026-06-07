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
  let timeStr = $state("00:00:00.000");
  let posToId = $state(new Map<string, number>());

  // 挑战模式
  let challengeMode = $state(false);
  let difficulty = $state<"easy" | "medium" | "hard">("easy");
  let showFailDialog = $state(false);
  let challengeTime = $state(0);
  let remainingTimeStr = $state("");

  const DIFFICULTIES = {
    easy: { label: "简单", size: 3, timeLimit: 60_000 },
    medium: { label: "中等", size: 4, timeLimit: 120_000 },
    hard: { label: "困难", size: 5, timeLimit: 300_000 },
  };

  let timeLimit = $derived(DIFFICULTIES[difficulty].timeLimit);

  let blocks: BlocksConfig = $state([]);
  function resize(){
    const cfg: BlocksConfig = [];
    posToId.clear();
    for (let x = 0; x < size; x++) {
      for (let y = 0; y < size; y++) {
        const id = y * size + x;
        cfg[id] = { x, y };
        posToId.set(`${x},${y}`, id);
      }
    }
    cfg[size*size-1] = {
      x: size - 1,
      y: size - 1,
    };
    posToId.set(`${size - 1},${size - 1}`, -1);
    console.log("resize", cfg)
    blocks = cfg;
  }
  $effect.pre(() => {
    resize();
  });

  function getBlock(id: number) {
    return blocks[id];
  }

  function getId(pos: Pos) {
    const { x, y } = pos;
    return posToId.get(`${x},${y}`) ?? -1;
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
      posToId.set(`${prevX},${prevY}`, currentId);

      if (currentY == srcY && currentX == srcX) {
        blocks[size * size - 1] = { x: srcX, y: srcY };
        posToId.set(`${srcX},${srcY}`, -1);

        break;
      }
    }
  }

  function shuffle(onDone?: () => void) {
    shuffling = true;
    start = false;
    timeStr = "00:00:00.000";

    const times = size * size * randInt(1, 10);

    const delay = showShuffle ? 20 : 0;

    interval(
      times,
      delay,
      (i) => {
        const { x, y } = blocks[size * size - 1];

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
        onDone?.();
      },
    );
  }

  function onChallengeTimeUp() {
    start = false;
    showFailDialog = true;
  }

  function startChallenge() {
    const diff = DIFFICULTIES[difficulty];
    size = diff.size;
    resize();
    showShuffle = false;
    showFailDialog = false;
    showWinDialog = false;
    start = false;
    shuffle(() => { start = true; });
  }

  function handleDifficultyChange(d: "easy" | "medium" | "hard") {
    difficulty = d;
    if (challengeMode) {
      const diff = DIFFICULTIES[difficulty];
      size = diff.size;
      resize();
      showFailDialog = false;
      showWinDialog = false;
    }
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
    }

    if (checkIsWin()) {
      start = false;
      if (challengeMode) {
        const remaining = Math.max(0, timeLimit - challengeTime);
        const h = String(Math.floor(remaining / 3600000)).padStart(2, "0");
        const m = String(Math.floor((remaining / 60000) % 60)).padStart(2, "0");
        const s = String(Math.floor((remaining / 1000) % 60)).padStart(2, "0");
        const ms = String(Math.floor(remaining % 1000)).padStart(3, "0");
        remainingTimeStr = `${h}:${m}:${s}.${ms}`;
      }
      showWinDialog = true;
    }
  }

  function checkIsWin() {
    const empty = blocks[size * size-1];
    return (
      empty.x == size - 1 && 
      empty.y == size - 1 && 
      !blocks.find((pos, i) => pos.y * size + pos.x !== i)
    );
  }
</script>

<Header />

<div class="container flex flex-wrap select-none m-1 max-w-max">
  <BigBorder {size}>
    {#each { length: size * size }, id}
      <Block
        bind:start
        {id}
        {size}
        {blocks}
        {showNum}
        {handleClick}
        {shuffling}
      />
    {/each}
  </BigBorder>

  <div class="flex flex-col m-1 gap-1 w-40">
    <Label>
      挑战模式
      <Switch bind:checked={challengeMode} class="ml-auto mr-0" />
    </Label>

    {#if challengeMode}
      <div class="flex gap-1">
        {#each Object.entries(DIFFICULTIES) as [key, diff]}
          <button
            class="px-2 py-1 rounded text-sm cursor-pointer disabled:cursor-not-allowed disabled:opacity-40 {difficulty === key ? 'bg-primary text-primary-foreground' : 'bg-gray-200 hover:bg-gray-300'}"
            onclick={() => handleDifficultyChange(key as 'easy' | 'medium' | 'hard')}
            disabled={start}
          >
            {diff.label}
          </button>
        {/each}
      </div>
    {:else}
      <Label class="text-nowrap">
        大小
        <Input type="number" bind:value={size} class="w-max" />
      </Label>

      <Label>
        显示打乱过程
        <Switch
          bind:checked={showShuffle}
          class="ml-auto mr-0"
          disabled={shuffling}
        />
      </Label>
    {/if}

    <Label>
      显示数字
      <Switch bind:checked={showNum} class="ml-auto mr-0" />
    </Label>

    <Button
      variant="outline"
      class="bg-gray-200 hover:bg-gray-300 border-gray-400 "
      onclick={() => {
        if (challengeMode && start) {
          start = false;
        } else if (challengeMode) {
          startChallenge();
        } else {
          shuffle();
        }
      }}
      disabled={shuffling}
    >
      {challengeMode && start ? "暂停" : challengeMode ? "开始挑战" : "打乱"}
    </Button>
  </div>

  <Timer
    bind:start
    bind:timeStr
    bind:time={challengeTime}
    type={challengeMode ? "countdown" : "stopwatch"}
    timeLimit={challengeMode ? timeLimit : 0}
    onTimeUp={onChallengeTimeUp}
  />

  <AlertDialog.Root bind:open={showWinDialog}>
    <AlertDialog.Content>
      <AlertDialog.Header class="m-auto">
        {challengeMode ? "挑战成功!" : "胜利!"}
      </AlertDialog.Header>
      <AlertDialog.Description>
        {#if challengeMode}
           梅露露酱在还剩{remainingTimeStr}的时候就在难度「{DIFFICULTIES[difficulty].label}」下通关了!
        {:else}
          梅露露酱仅用{timeStr}就结束了比赛!
        {/if}
        <br />大魔女十分欣慰~
      </AlertDialog.Description>
      <AlertDialog.Footer>
        <AlertDialog.Action
          onclick={() => (showWinDialog = false)}
          class="m-auto"
        >
          继续
        </AlertDialog.Action>
      </AlertDialog.Footer>
    </AlertDialog.Content>
  </AlertDialog.Root>

  <AlertDialog.Root bind:open={showFailDialog}>
    <AlertDialog.Content>
      <AlertDialog.Header class="m-auto text-red-600">挑战失败</AlertDialog.Header>
      <AlertDialog.Description>
        梅露露酱未能在「{DIFFICULTIES[difficulty].label}」难度规定时间内完成复原<br />
        不要气馁，再来一次吧!
      </AlertDialog.Description>
      <AlertDialog.Footer>
        <AlertDialog.Action
          onclick={() => { showFailDialog = false; }}
          class="m-auto"
        >
          确定
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
