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
  
    async function fetchBitcoinData() {
      try {
        let response = await fetch("https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=EUR&apikey=demo");
        if (!response.ok) throw new Error("Failed to fetch Bitcoin data");
        let data = await response.json();
        let timeSeries = data["Time Series (Digital Currency Daily)"];
        if (!timeSeries) throw new Error("Invalid API response format");
  
        let dates = Object.keys(timeSeries).sort((a, b) => new Date(b) - new Date(a));
        let todayDate = dates[0]; // Latest data (today)
        let yesterdayDate = dates[1]; // Previous day (yesterday)
  
        let todayData = timeSeries[todayDate];
        let yesterdayData = timeSeries[yesterdayDate];
  
        console.log("Today's date:", todayDate); // Debugging log
        console.log("Yesterday's date:", yesterdayDate); // Debugging log
        console.log("Today's data:", todayData); // Debugging log
        console.log("Yesterday's data:", yesterdayData); // Debugging log
  
        // Dynamically set column headers from the data
        columnHeaders = Object.keys(todayData).map(key => {
          // Remove the numeric part to create a user-friendly header
          switch(key) {
            case '1. open': return 'Open';
            case '2. high': return 'High';
            case '3. low': return 'Low';
            case '4. close': return 'Close';
            case '5. volume': return 'Volume';
            default: return key;
          }
        });
  
        // Prepare the data
        tableData = [
          [todayDate, ...Object.values(todayData)],
          [yesterdayDate, ...Object.values(yesterdayData)]
        ];
  
        // Compare values for each cell and highlight if change > 20%
        tableData.forEach((row, rowIndex) => {
          row.forEach((cell, colIndex) => {
            if (rowIndex === 1 && colIndex > 0) { // Skip the first column (date)
              let todayValue = parseFloat(tableData[0][colIndex]);
              let yesterdayValue = parseFloat(cell);
  
              // Calculate percentage change
              let change = ((todayValue - yesterdayValue) / yesterdayValue) * 100;
  
              // Check if the change is greater than 20%
              if (Math.abs(change) > 20) {
                row[colIndex] = { value: cell, highlighted: true };
              } else {
                row[colIndex] = { value: cell, highlighted: false };
              }
            }
          });
        });
  
        console.log("Updated tableData:", tableData); // Debugging log
  
        // After table data is fetched and processed, initialize or update the table
        initializeTable();
        
      } catch (err) {
        console.error("Bitcoin API fetch error:", err);
      }
    }
  
    // Ensure Handsontable is initialized after updates
    afterUpdate(() => {
      initializeTable();
    });
  
    onMount(() => {
      fetchBitcoinData(); // Fetch Bitcoin data when component is mounted
    });
  </script>
  
  <div bind:this={container}></div>
  
  <style>
    .highlighted {
      background-color: green;
      color: white;
      font-weight: bold;
    }
  </style>
  
