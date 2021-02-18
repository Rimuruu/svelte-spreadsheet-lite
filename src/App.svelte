<script >
  import SpreadsheetLite from "src/components/SpreadsheetLite.svelte";

  $: header = [
    { name: "Name", width: 200 },
    { name: "Age", width: 60 },
    { name: "Gender", width: 60 }
  ];
  $: data = [
    ["Yamada Taro", "12", "F"],
    ["Yamada Hanako", "12", "M"],
    ["Sato Jiro", "12", "M"],
    ["Sato Hanako", "12", "M"]
  ];
 
  function addRow() {
    data = [...data, ["", "", ""]];
  }
</script>
<style>
.center {
  margin: auto;
  width: 50%;
  border: 2px solid grey;
  padding: 10px;
  background-color:white;
}


</style>

<div class="center">
<h1>Svelte spreadsheet lite</h1> 
<table class="table table-bordered">
  <thead>
    <tr>
      {#each header as label}
        <th>{label.name}</th>
      {/each}
    </tr>
  </thead>
  <tbody>
    {#each data as row}
      <tr>
        {#each row as col, ci}
          {#if ci == 0}
            <th scope="row">{col}</th>
          {:else}
            <td>{col}</td>
          {/if}
        {/each}
      </tr>
    {/each}
  </tbody>
</table>



<SpreadsheetLite bind:data bind:header />
<button on:click={addRow}>Add Row</button>
<div>based on : <a href="https://github.com/anydown/vue-spreadsheet-lite">@anydown/vue-spreadsheet-lite</a> </div>
</div>
