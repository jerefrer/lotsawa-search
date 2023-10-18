<script>
  import { slide } from "svelte/transition";

  export let line;

  let opened = false;

  const toggleOpened = function () {
    opened = !opened;
  };
</script>

<div class="line flex pl-4">
  <div class="flex-1">
    {#if line.previousTibetan && opened}
      <div transition:slide class="bg-[#f6ebeb] p-4 pr-0 pt-1" style="margin-left: -16px">
        <div class="tibetan text-lg leading-relaxed mt-4">
          {@html line.previousTibetan}
        </div>
        <div class="french leading-snug !mt-1">
          {line.previousFrench}
        </div>
      </div>
    {/if}
    <div
      class="tibetan leading-relaxed mt-4"
      class:text-lg={!line.smallLetters}
      class:text-sm={line.smallLetters}
    >
      {@html line.tibetan}
    </div>
    <div
      class="french leading-snug !mt-1"
      class:text-sm={line.smallLetters}
      class:italic={line.smallLetters}
    >
      {line.french}
    </div>
    {#if line.nextTibetan && opened}
      <div transition:slide  class="bg-[#f6ebeb] mt-4 p-4 pr-0 pt-1" style="margin-left: -16px; margin-bottom: -10px;">
        <div class="tibetan text-lg leading-relaxed mt-4">
          {@html line.nextTibetan}
        </div>
        <div class="french leading-snug !mt-1">
          {line.nextFrench}
        </div>
      </div>
    {/if}
  </div>
  {#if line.previousTibetan}
    <button
      on:click={toggleOpened}
      class="flex flex-col right text-gray-400 bg-gray-100 hover:text-gray-600 hover:bg-gray-200 py-2"
    >
      <svg
        class:rotate-anticlockwise={opened}
        xmlns="http://www.w3.org/2000/svg"
        width="16"
        height="16"
        viewBox="0 0 24 24"
      >
        <path
          fill="currentColor"
          d="M11 20V7.825l-5.6 5.6L4 12l8-8l8 8l-1.4 1.425l-5.6-5.6V20h-2Z"
        />
      </svg>
      <div class="flex-1"></div>
      <svg
        class:rotate-clockwise={opened}
        xmlns="http://www.w3.org/2000/svg"
        width="16"
        height="16"
        viewBox="0 0 24 24"
      >
        <path
          fill="currentColor"
          d="M11 4v12.175l-5.6-5.6L4 12l8 8l8-8l-1.4-1.425l-5.6 5.6V4h-2Z"
        />
      </svg>
    </button>
  {/if}
</div>

<style>
  .line {
    border-bottom: 1px solid lightgray;
  }
  .line:last-child {
    border-bottom: none;
    border-bottom-left-radius: 7.1px;
    border-bottom-right-radius: 7.1px;
  }
  .line:last-child div:last-child {
    border-bottom-left-radius: 7.1px;
  }
  .line:last-child button {
    border-bottom-right-radius: 7.1px;
  }
  .line > div {
    padding-bottom: 10px;
  }
  .line .tibetan {
    font-family: "DDC Uchen";
    color: #954a29;
  }

  svg {
    transition: 0.4s ease-in-out;
  }
  svg.rotate-clockwise {
    transform: rotate(180deg);
  }
  svg.rotate-anticlockwise {
    transform: rotate(-180deg);
  }
</style>
