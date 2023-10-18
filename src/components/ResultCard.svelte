<script>
  import ResultCardLine from "./ResultCardLine.svelte";

  export let prayer;
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class="prayer rounded-lg bg-red-50 border border-gray-300"
>
  <h2 class="flex items-center p-4 leading-4 bg-red-100 rounded-t-lg border-b border-gray-300">
    <a
      href={prayer.url}
      target="_blank"
      class="italic no-underline !text-gray-600 hover:!text-gray-800 hover:underline"
    >
      {prayer.name}
    </a>
  </h2>
  {#each prayer.lines as line (line.tibetan) }
    <ResultCardLine {line} />
  {/each}
</div>

<style>
  .line {
    padding-bottom: 20px;
    border-bottom: 1px solid lightgray;
  }
  
  .line .line:last-child {
    padding-bottom: 0;
    border-bottom: 0;
  }
</style>