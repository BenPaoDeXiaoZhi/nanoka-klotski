<script lang="ts">
  import { onMount } from "svelte";

  type Props = {
    start: boolean;
    timeStr?: string;
    time?: number;
    type?: "stopwatch" | "countdown";
    timeLimit?: number;
    onTimeUp?: () => void;
  };

  let {
    start = $bindable(false),
    time = $bindable(0),
    timeStr = $bindable("00:00:00.000"),
    type = "stopwatch",
    timeLimit = 0,
    onTimeUp,
  }: Props = $props();
  let startTime = $state(Date.now());
  $effect(() => {
    if (start) {
      startTime = Date.now();
      time = 0;
    }
  });
  onMount(() => {
    const id = setInterval(() => {
      if (start) {
        time = Date.now() - startTime;

        if (type === "countdown" && time >= timeLimit) {
          start = false;
          time = timeLimit;
          onTimeUp?.();
        }
      }
    }, 4);
    return () => {
      clearInterval(id);
    };
  });

  let displayMs = $derived(
    type === "countdown" ? Math.max(0, timeLimit - time) : time,
  );

  let hour = $derived(
    String(Math.floor(displayMs / 1000 / 3600)).padStart(2, "0"),
  );
  let minute = $derived(
    String(Math.floor((displayMs / 1000 / 60) % 60)).padStart(2, "0"),
  );
  let second = $derived(
    String(Math.floor((displayMs / 1000) % 60)).padStart(2, "0"),
  );
  let millSecond = $derived(
    String(Math.floor(displayMs % 1000)).padStart(3, "0"),
  );
</script>

<div
  class="bg-gray-200 border border-gray-400 p-2 rounded-md text-center text-xl h-fit {type === 'countdown' && displayMs < timeLimit * 0.25 ? 'text-red-600' : ''}"
>
  {timeStr}
</div>