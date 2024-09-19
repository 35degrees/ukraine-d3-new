<script>
  export let items = []
  export let selectedValue = 'personnel'
</script>

<p>Select Category</p>
<select name="name" bind:value={selectedValue}>
  {#each items as i}
  <option value={i}>{i}</option>
  {/each}
</select>

<style>
  p {
    display: inline-block;
    margin-right: 5px;
    font-size: 14px;
  }

  select {
    margin-bottom: 30px;
    margin-top: 0;
  }
</style>