#
# Svelte spreadsheet-lite

A spreadsheet component for svelte based on @anydown/vue-spreadsheet-lite.
I did during my intership.

```
import SpreadsheetLite from "src/components/SpreadsheetLite.svelte";

data = ["HEADER","HEADER","HEADER"];
data = [["CELL","CELL","CELL"]];
....
<SpreadsheetLite bind:data bind:header />
```
