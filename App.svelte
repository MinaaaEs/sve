<script>
  import { onMount } from "svelte";
  import Handsontable from "handsontable";
  import "handsontable/dist/handsontable.full.css";

  let container;
  let hot;
  let isToggled = false; // Toggle state

  // Placeholder for transposed table data
  let data1 = [];
  let data2 = [];
  let columnHeaders = []; // Column headers to be dynamically populated

  // Function to fetch Bitcoin data from the API
  async function fetchBitcoinData() {
    const response = await fetch(
      "https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=EUR&apikey=demo"
    );
    const data = await response.json();
    const timeSeries = data["Time Series (Digital Currency Daily)"];

    // Extract the first and second elements from the time series
    const dates = Object.keys(timeSeries);
    if (dates.length >= 2) {
      const firstDate = dates[0];
      const secondDate = dates[1];

      // Dynamically generate column headers (with Date first)
      columnHeaders = generateColumnHeaders(timeSeries[firstDate]);

      // Format data for each table (transposed) with Date as the first column
      data1 = formatData(timeSeries[firstDate], firstDate);
      data2 = formatData(timeSeries[secondDate], secondDate);

      // Initialize Handsontable with the first dataset (default table)
      initializeTable(container, data1);
    }
  }

  // Generate column headers dynamically, placing "Date" first
  function generateColumnHeaders(data) {
    const headers = [
      "Date", 
      "Open", 
      "High", 
      "Low", 
      "Close", 
      "Volume"
    ];
    return headers;
  }

  // Format the data for each table (Transpose it) with Date as the first column
  function formatData(data, date) {
    return [
      [date, data["1. open"], data["2. high"], data["3. low"], data["4. close"], data["5. volume"]] // Date is the first column
    ];
  }

  // Initialize the Handsontable instance with a given dataset
  function initializeTable(container, tableData) {
    hot = new Handsontable(container, {
      data: tableData, // No need to transpose now; we use the formatted data as-is
      colHeaders: columnHeaders, // Column headers dynamically generated
      rowHeaders: false, // No row headers
      stretchH: "all",
      width: "100%",
      height: "100%",
      licenseKey: "non-commercial-and-evaluation",
      columns: [{ type: "text" }, { type: "numeric" }, { type: "numeric" }, { type: "numeric" }, { type: "numeric" }, { type: "numeric" }],
      afterChange(changes, source) {
        if (source !== 'loadData' && changes) {
          applyCyanColoring(); // Reapply cyan coloring on data change
          applyVioletColoring(changes); // Apply violet coloring if values change to numeric
        }
      },

      beforeChange(changes, source) {
        changes.forEach(([row, prop, oldValue, newValue], index) => {
          // Check if the new value is a valid number, if not revert to old value
          if (newValue !== "" && isNaN(newValue)) {
            changes[index][3] = oldValue; // Revert to the old value
            alert("Invalid input! Only numeric values are allowed.");
          }
          else {
            alert("changes saved.");
          }
        });
      },

      // Apply cyan coloring immediately after the table is rendered
      afterRender() {
        applyCyanColoring(); // Ensure cyan coloring is applied when the table is first rendered
      }
    });
  }

  // Apply cyan coloring to cells where the difference between tables is greater than 30
  function applyCyanColoring() {
    if (!hot) return; // Ensure hot is defined before continuing

    // Only apply cyan coloring if the data is available
    if (data1 && data2) {
      for (let i = 1; i < data1[0].length; i++) { // Start from index 1 since index 0 is the Date column
        const val1 = data1[0][i];
        const val2 = data2[0][i];

        if (Math.abs(val1 - val2) > 30) {
          const cell1 = hot.getCell(0, i); // Table 1 cell
          const cell2 = hot.getCell(1, i); // Table 2 cell

          if (cell1) cell1.style.backgroundColor = "cyan";
          if (cell2) cell2.style.backgroundColor = "cyan";
        }
      }
    }
  }

  // Apply violet coloring to cells where the value is changed to a numeric value
  function applyVioletColoring(changes) {
    if (!hot) return; // Ensure hot is defined before continuing

    changes.forEach(([row, prop, oldValue, newValue]) => {
      // If the value has changed and the new value is numeric, color the cell in violet
      if (!isNaN(newValue) && oldValue !== newValue) {
        const col = hot.propToCol(prop);
        const cell = hot.getCell(row, col);
        if (cell) {
          // Apply violet background color to changed cells
          cell.style.backgroundColor = "violet";
        }
      }
    });
  }

  // Toggle function to switch between Table 1 and Table 2
  function toggleTable() {
    isToggled = !isToggled;
    hot.loadData(isToggled ? data2 : data1); // Load the appropriate table data
    hot.render(); // Force Handsontable to re-render
    applyCyanColoring(); // Reapply cyan coloring after switching tables
  }

  // Fetch Bitcoin data when the component mounts
  onMount(() => {
    fetchBitcoinData();
  });
</script>

<div class="layout">
  <!-- Left Panel -->
  <div class="left-panel">
    <h3>Options</h3>
    <button class="toggle-button" on:click={toggleTable}>
      {isToggled ? "Show Table 1" : "Show Table 2"}
    </button>
  </div>

  <!-- Middle Panel (Table) -->
  <div class="middle-panel">
    <div bind:this={container} class="table-container"></div>
  </div>

  <!-- Right Panel -->
  <div class="right-panel">
    <h3>Additional Info</h3>
    <p>Display extra content here if needed.</p>
  </div>
</div>

<style>
  /* Layout for the entire page */
  .layout {
    display: grid;
    grid-template-columns: 1fr 3fr 1fr; /* Left (1), Middle (3), Right (1) */
    grid-gap: 20px;
    padding: 20px;
    height: 100vh;
  }

  /* Left Panel Styling */
  .left-panel {
    background-color: #f4f4f9;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .left-panel h3 {
    font-size: 1.5em;
    color: #333;
  }

  .toggle-button {
    padding: 10px 20px;
    font-size: 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    width: 100%;
  }

  .toggle-button:hover {
    background-color: #0056b3;
  }

  /* Middle Panel (Table) Styling */
  .middle-panel {
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }

  .table-container {
    width: 100%;
    max-width: 800px;
    height: 400px;
  }

  /* Right Panel Styling */
  .right-panel {
    background-color: #f4f4f9;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .right-panel h3 {
    font-size: 1.5em;
    color: #333;
  }
</style>


