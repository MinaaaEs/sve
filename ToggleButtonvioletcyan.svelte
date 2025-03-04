<script>
    import { onMount } from "svelte";
    import Handsontable from "handsontable";
    import "handsontable/dist/handsontable.full.css";
  
    let container;
    let hot;
    let isToggled = false; // Toggle state
  
    // Two 3x3 datasets
    let data1 = [
      [5, 10, 15],
      [40, 35, 25],
      [60, 5, 45]
    ];
  
    let data2 = [
      [20, 50, 10],
      [30, 25, 40],
      [5, 60, 35]
    ];
  
    // Store last valid numeric values separately for each dataset
    let lastValidData1 = JSON.parse(JSON.stringify(data1));
    let lastValidData2 = JSON.parse(JSON.stringify(data2));
  
    // Store changed cells to apply violet coloring
    let changedCells = new Set();
    let savedChanges = [];
  
    // Function to check if value is numeric (but allow empty)
    function isNumeric(value) {
      return value === "" || (!isNaN(parseFloat(value)) && isFinite(value));
    }
  
    // Custom renderer to color cells if the difference between tables > 30 or if updated
    function customRenderer(instance, td, row, col, prop, value, cellProperties) {
      Handsontable.renderers.TextRenderer.apply(this, arguments); // Default rendering
  
      const comparisonData = isToggled ? data1 : data2; // Get the dataset NOT currently displayed
      const comparisonValue = comparisonData[row]?.[col] ?? null;
  
      // Apply violet background if the cell value was updated
      const cellKey = `${row}-${col}`;
      if (changedCells.has(cellKey)) {
        td.style.backgroundColor = "violet"; // Violet always takes precedence
      } else if (comparisonValue !== null && value !== "") {
        const diff = Math.abs(value - comparisonValue);
        if (diff > 30) {
          td.style.backgroundColor = "cyan";
        } else {
          td.style.backgroundColor = ""; // Reset background if condition isn't met
        }
      } else {
        td.style.backgroundColor = ""; // Reset background if empty
      }
    }
  
    // Handle changes and prevent non-numeric values (except empty)
    function handleChange(changes, source) {
      if (source === "loadData") return; // Ignore initial data load
  
      let currentData = isToggled ? data2 : data1; // Get current dataset
      let lastValidData = isToggled ? lastValidData2 : lastValidData1; // Get last valid dataset
  
      changes.forEach(([row, col, oldValue, newValue]) => {
        if (!isNumeric(newValue)) {
          console.warn(`Invalid input at row ${row}, col ${col}: "${newValue}". Resetting to last valid number.`);
  
          // Restore last valid number immediately
          hot.setDataAtCell(row, col, lastValidData[row][col]);
        } else {
          // If empty, allow it
          if (newValue === "") {
            currentData[row][col] = "";
          } else {
            // Update last valid number and mark cell as changed for violet coloring
            lastValidData[row][col] = parseFloat(newValue);
            currentData[row][col] = parseFloat(newValue);
            const cellKey = `${row}-${col}`;
            changedCells.add(cellKey);
            savedChanges.push({ row, col, oldValue, newValue });
          }
        }
      });
    }
  
    // Initialize Handsontable
    onMount(() => {
      hot = new Handsontable(container, {
        data: data1, // Start with the first dataset
        colHeaders: ["A", "B", "C"],
        rowHeaders: true,
        licenseKey: "non-commercial-and-evaluation",
        columns: [{ type: "numeric" }, { type: "numeric" }, { type: "numeric" }],
        cells(row, col) {
          return { renderer: customRenderer };
        },
        afterChange: handleChange
      });
    });
  
    // Toggle function to switch between datasets
    function toggleTable() {
      isToggled = !isToggled;
      hot.loadData(isToggled ? data2 : data1); // Load new data
      hot.render(); // Ensure table updates
    }
  </script>
  
  <!-- Toggle Button -->
  <button on:click={toggleTable}>
    {isToggled ? "Show Table 1" : "Show Table 2"}
  </button>
  
  <!-- Table container -->
  <div bind:this={container}></div>
  
  <style>
    button {
      padding: 10px;
      font-size: 16px;
      margin-bottom: 10px;
      cursor: pointer;
    }
  </style>
  