<script>
  export let item;
</script>

<div class="customItem">
  <span
    class="fi fi-{item.countryCode}"
  />
</div>

<style is:global>
  .svelte-select-list {
    scrollbar-width: 1px !important;
  }
</style>