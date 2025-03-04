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
  
    // Store last valid numeric values
    let lastValidData1 = JSON.parse(JSON.stringify(data1));
    let lastValidData2 = JSON.parse(JSON.stringify(data2));
  
    // Function to check if value is numeric
    function isNumeric(value) {
      return !isNaN(value) && value !== null && value !== "";
    }
  
    // Custom renderer to color cells if the difference between tables > 30
    function customRenderer(instance, td, row, col, prop, value, cellProperties) {
      Handsontable.renderers.TextRenderer.apply(this, arguments); // Default rendering
  
      const comparisonData = isToggled ? data1 : data2; // Get the dataset NOT currently displayed
      const comparisonValue = comparisonData[row]?.[col] ?? null;
  
      if (comparisonValue !== null) {
        const diff = Math.abs(value - comparisonValue);
        if (diff > 30) {
          td.style.backgroundColor = "cyan";
        } else {
          td.style.backgroundColor = ""; // Reset background if condition isn't met
        }
      }
    }
  
    // Handle changes and prevent non-numeric values
    function handleChange(changes, source) {
      if (source === "loadData") return; // Ignore initial data load
  
      changes.forEach(([row, col, oldValue, newValue]) => {
        if (!isNumeric(newValue)) {
          console.warn(`Invalid input at row ${row}, col ${col}: "${newValue}". Resetting to last valid number.`);
  
          // Restore last valid number
          if (isToggled) {
            hot.setDataAtCell(row, col, lastValidData2[row][col]);
          } else {
            hot.setDataAtCell(row, col, lastValidData1[row][col]);
          }
        } else {
          // Store new valid number
          if (isToggled) {
            lastValidData2[row][col] = newValue;
          } else {
            lastValidData1[row][col] = newValue;
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
  