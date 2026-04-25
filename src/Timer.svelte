<script lang="ts">
  import { onMount } from "svelte";

  type Props = {
    start: boolean;
    timeStr?: string;
    time?: number;
  };

  let {
    start = $bindable(false),
    time = $bindable(0),
    timeStr = $bindable(""),
  }: Props = $props();
  let startTime = $state(Date.now());
  $effect(() => {
    if (start) {
      startTime = Date.now();
    }
  });
  onMount(() => {
    const id = setInterval(() => {
      if (start) {
        time = Date.now() - startTime;
        timeStr = `${hour}:${minute}:${second}.${millSecond}`;
      }
    }, 4);
    return () => {
      clearInterval(id);
    };
  });

  let hour = $derived(String(Math.floor(time / 1000 / 3600)).padStart(2, "0"));
  let minute = $derived(
    String(Math.floor((time / 1000 / 60) % 60)).padStart(2, "0"),
  );
  let second = $derived(
    String(Math.floor((time / 1000) % 60)).padStart(2, "0"),
  );
  let millSecond = $derived(String(Math.floor(time % 1000)).padEnd(3, "0"));
  timeStr = `00:00:00.000`;
</script>

<div>
  {timeStr}
</div>

<style>
  div {
    background-color: #e0e2e4;
    border: 1px #abacad solid;
    border-radius: 1em;
    padding: 0.5em;
    text-align: center;
    font-size: 1.5em;
    height: fit-content;
  }
</style>
