<script>
  import { onMount, afterUpdate, createEventDispatcher } from 'svelte';
  import Handsontable from 'handsontable';
  import 'handsontable/dist/handsontable.full.css';

  export let rangeValues; // Receive from App.svelte
  let container;
  let hot;

  const dispatch = createEventDispatcher(); // Send data back to App.svelte

  let changedCells = new Set();
  let savedChanges = [];

  // Generate years dynamically from rangeValues
  $: years = Array.from(
    { length: rangeValues[1] - rangeValues[0] + 1 },
    (_, i) => (rangeValues[0] + i).toString()
  );

  // Generate random initial data
  $: data = Array.from({ length: 5 }, () =>
    years.map(() => Math.floor(Math.random() * 100))
  );

  function initializeTable() {
    if (hot) {
      // Update table dynamically instead of recreating it
      hot.updateSettings({
        data: data,
        colHeaders: years,
      });
    } else {
      // Initialize table only once
      hot = new Handsontable(container, {
        data: data,
        colHeaders: years,
        rowHeaders: true,
        stretchH: 'all',
        width: '100%',
        height: '100%',
        licenseKey: 'non-commercial-and-evaluation',

        beforeChange(changes, source) {
          changes.forEach(([row, prop, oldValue, newValue], index) => {
            if (isNaN(newValue)) {
              changes[index][3] = oldValue; // Revert non-numeric input
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

  // Ensure table updates when rangeValues change
  afterUpdate(() => {
    initializeTable();
  });

  onMount(() => {
    initializeTable();
  });
</script>

<div bind:this={container}></div>
