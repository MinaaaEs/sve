<script>
    import { onMount, afterUpdate, createEventDispatcher } from 'svelte';
    import Handsontable from 'handsontable';
    import 'handsontable/dist/handsontable.full.css';
  
    export let tableData = [];
    export let columnHeaders = [];
    export let referenceData = []; // Reference dataset for comparison
    export let highlightedCells = new Set();  // Highlighted cells passed from App.svelte
  
    let container;
    let hot;
  
    const dispatch = createEventDispatcher();
    let changedCells = new Set();
    let savedChanges = [];
  
    // 🚀 Custom Renderer: Highlights differences & user changes
    function customRenderer(instance, td, row, col, prop, value, cellProperties) {
      Handsontable.renderers.TextRenderer.apply(this, arguments);
  
      const cellKey = `${row}-${col}`;
      let originalValue = referenceData[row]?.[col];
  
      // Debug log for every cell render
      if (originalValue !== undefined && value !== originalValue) {
        td.style.backgroundColor = 'lightcoral'; // 🔴 Highlight differences
        console.log(`⚠️ Difference detected at (${row}, ${col})`);
      } else {
        td.style.backgroundColor = ''; // Reset if values match
      }
  
      // Apply user-edit highlight (pink)
      if (changedCells.has(cellKey)) {
        td.style.backgroundColor = 'pink'; // 🟣 Highlight user edits
      }
  
      // Apply highlight based on highlightedCells Set
      if (highlightedCells.has(cellKey)) {
        td.style.backgroundColor = 'violet'; // 🟢 Highlight significant cells
      }
    }
  
    function initializeTable() {
      if (hot) {
        console.log("🔄 Updating Handsontable...");
        hot.updateSettings({
          data: tableData,
          colHeaders: columnHeaders,
          columns: columnHeaders.map(() => ({ renderer: customRenderer })),
        });
        hot.render();
      } else {
        console.log("🚀 Initializing Handsontable...");
        hot = new Handsontable(container, {
          data: tableData,
          colHeaders: columnHeaders,
          rowHeaders: true,
          stretchH: 'all',
          width: '100%',
          height: '100%',
          licenseKey: 'non-commercial-and-evaluation',
          columns: columnHeaders.map(() => ({ renderer: customRenderer })),
  
          beforeChange(changes, source) {
            console.log("🛠️ Before Change:", changes, "Source:", source);
            changes.forEach(([row, prop, oldValue, newValue], index) => {
              if (isNaN(newValue)) {
                changes[index][3] = oldValue;
              }
            });
          },
  
          afterChange(changes, source) {
            if (source !== 'loadData' && changes) {
              console.log("✅ Changes detected:", changes);
              changes.forEach(([row, prop, oldValue, newValue]) => {
                if (oldValue !== newValue) {
                  const column = hot.propToCol(prop);
                  const cellKey = `${row}-${column}`;
                  changedCells.add(cellKey);
                  savedChanges.push({ row, column, oldValue, newValue });
                  console.log(`✏️ User changed (${row}, ${column}) from ${oldValue} → ${newValue}`);
                }
              });
  
              dispatch("updateChanges", { savedChanges });
              hot.render();
            }
          }
        });
      }
    }
  
    afterUpdate(() => {
      console.log("🔄 After Update: Re-rendering table...");
      initializeTable();
    });
  
    onMount(() => {
      console.log("🚀 Component Mounted: Initializing Table...");
      initializeTable();
    });
  </script>
  
  <div bind:this={container}></div>
  
  <style>
    /* Optionally define highlight colors or styles */
    .highlight {
      background-color: violet !important;
      color: red !important;
      font-weight: bold !important;
    }
  </style>
  