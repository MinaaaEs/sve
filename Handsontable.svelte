<script>
  import { onMount, afterUpdate, createEventDispatcher } from 'svelte';
  import Handsontable from 'handsontable';
  import 'handsontable/dist/handsontable.full.css';

  export let tableData = [];
  export let columnHeaders = [];
  let container;
  let hot;

  const dispatch = createEventDispatcher();
  let changedCells = new Set();
  let savedChanges = [];

  function initializeTable() {
    if (hot) {
      hot.updateSettings({
        data: tableData,
        colHeaders: columnHeaders,
      });
    } else {
      hot = new Handsontable(container, {
        data: tableData,
        colHeaders: columnHeaders,
        rowHeaders: true,
        stretchH: 'all',
        width: '100%',
        height: '100%',
        licenseKey: 'non-commercial-and-evaluation',

        beforeChange(changes, source) {
          changes.forEach(([row, prop, oldValue, newValue], index) => {
            if (isNaN(newValue)) {
              changes[index][3] = oldValue;
            }
          });
        },

        afterChange(changes, source) {
          if (source !== 'loadData' && changes) {
            changes.forEach(([row, prop, oldValue, newValue]) => {
              if (oldValue !== newValue) {
                const column = hot.propToCol(prop);
                const cellKey = `${row}-${column}`;
                changedCells.add(cellKey);
                savedChanges.push({ row, column, oldValue, newValue });
              }
            });

            dispatch("updateChanges", { savedChanges });

            // Apply pink background to changed cells
            hot.addHookOnce('afterRender', () => {
              changedCells.forEach((cellKey) => {
                const [row, column] = cellKey.split('-').map(Number);
                const cell = hot.getCell(row, column);
                if (cell) cell.style.backgroundColor = 'pink';
              });
            });

            hot.render();
          }
        }
      });
    }
  }

  afterUpdate(() => {
    initializeTable();
  });

  onMount(() => {
    initializeTable();
  });
</script>

<div bind:this={container}></div>

