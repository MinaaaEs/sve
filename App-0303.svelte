<script>
  import Handsontable from './Handsontable.svelte';
  import { onMount } from 'svelte';

  let classSelectValue = 'equity'; // Default selection
  let tableData = [];
  let columnHeaders = [];
  let error = null;
  let showTicker = false;
  let selectedCountry = null;
  let selectedPeriod = null;
  let countryDropdownOpen = false;
  let periodDropdownOpen = false;
  let isAfter = false; // Default state (Before)
  let previousClassValue = null;
  let hot = null;

//   function toggle() {
//   isAfter = !isAfter;
//   if (classSelectValue === 'bitcoin') {
//     fetchBitcoinData(); // Re-fetch Bitcoin data when toggling
//   }
// }

  const countries = [
    { name: "Germany", code: "de" },
    { name: "France", code: "fr" },
    { name: "Spain", code: "es" },
    { name: "Italy", code: "it" },
    { name: "Netherlands", code: "nl" }
  ];

  const periods = [
    { name: "Year", code: "Y" },
    { name: "Quarter", code: "Q" }
  ];

  function toggleCountryDropdown() {
    countryDropdownOpen = !countryDropdownOpen;
    periodDropdownOpen = false;
  }

  function togglePeriodDropdown() {
    periodDropdownOpen = !periodDropdownOpen;
    countryDropdownOpen = false;
  }

  function selectCountry(country) {
    selectedCountry = country;
    countryDropdownOpen = false;
    fetchApiData(classSelectValue);
  }

  function selectPeriod(period) {
    selectedPeriod = period;
    periodDropdownOpen = false;
    fetchApiData(classSelectValue);
  }

  function toggle() {
    isAfter = !isAfter;
    if (classSelectValue === 'bitcoin') {
      updateBitcoinTable(); // Update table when toggling
    }
  }

  let bitcoinData = {}; // Store both before and after data separately
let latestDate = null;
let previousDate = null;

async function fetchBitcoinData() {
  try {
    let response = await fetch("https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=EUR&apikey=demo");
    if (!response.ok) throw new Error("Failed to fetch Bitcoin data");
    
    let data = await response.json();
    let timeSeries = data["Time Series (Digital Currency Daily)"];
    if (!timeSeries) throw new Error("Invalid API response format");

    let dates = Object.keys(timeSeries).sort((a, b) => new Date(b) - new Date(a));
    latestDate = dates[0];
    previousDate = dates[1];

    // Store both days of data separately
    bitcoinData.latest = timeSeries[latestDate];
    bitcoinData.previous = timeSeries[previousDate];

    // Dynamically create column headers based on the latest date
    columnHeaders = ["Date", ...Object.keys(timeSeries[latestDate]).map(key => key.replace(/\d+\. /, ''))];

    // Initially update the table
    updateBitcoinTable();
  } catch (err) {
    error = err.message;
    console.error("Bitcoin API fetch error:", error);
  }
}

function updateBitcoinTable() {
  if (!bitcoinData.latest || !bitcoinData.previous) return;

  // Select the correct data based on the toggle state
  let selectedData = isAfter ? bitcoinData.previous : bitcoinData.latest;
  let selectedDate = isAfter ? previousDate : latestDate;

  // Update column headers dynamically to reflect the correct set of values
  columnHeaders = ["Date", ...Object.keys(selectedData).map(key => key.replace(/\d+\. /, ''))];

  // Create table data row with the selected date and data
  tableData = [
    [selectedDate, ...Object.keys(selectedData).map(key => parseFloat(selectedData[key]) || selectedData[key])]
  ];

  // Highlight significant differences if needed
  highlightSignificantDifferences(selectedData, isAfter ? bitcoinData.latest : bitcoinData.previous);
}

let highlightedCells = new Set(); // Clear this at the start

function highlightSignificantDifferences(selectedData, referenceData) {
  highlightedCells.clear(); // Clear previous highlights

  if (!referenceData) return;

  // Iterate through the data to check the percentage change and highlight
  tableData = tableData.map((row, rowIndex) => {
    return row.map((cell, colIndex) => {
      // Skip non-relevant columns (e.g., Date column at index 0)
      if (colIndex === 0 || isNaN(cell)) return cell;

      // Get the reference value from the other day
      let referenceValue = parseFloat(referenceData[Object.keys(referenceData)[colIndex - 1]]);
      if (isNaN(referenceValue)) return cell;

      // Ensure cell is a valid number before calculating the percentage change
      let currentValue = parseFloat(cell);
      if (isNaN(currentValue)) return cell;

      // Calculate the percentage change
      let percentageChange = Math.abs((currentValue - referenceValue) / referenceValue);
      console.log(`Comparing cell ${rowIndex}-${colIndex}: Current Value: ${currentValue}, Reference Value: ${referenceValue}, Percentage Change: ${percentageChange}`);

      // Highlight cells if the change is greater than the threshold (e.g., 20%)
      if (percentageChange > 0.2) {
        console.log(`Highlighting cell ${rowIndex}-${colIndex}`);
        highlightedCells.add(`${rowIndex}-${colIndex}`); // Mark the cell to be highlighted
      }

      return cell;
    });
  });

  // Call applyHighlighting function after data is processed
  applyHighlighting();
}



function applyHighlighting() {
  // Create cellMeta structure to pass highlighting information to Handsontable
  let cellMeta = [];

  tableData.forEach((row, rowIndex) => {
    row.forEach((cell, colIndex) => {
      // If the cell is highlighted, apply the "highlight" class
      if (highlightedCells.has(`${rowIndex}-${colIndex}`)) {
        if (!cellMeta[rowIndex]) {
          cellMeta[rowIndex] = [];
        }
        // Assigning the class to cells that need highlighting
        cellMeta[rowIndex][colIndex] = { className: 'highlight' };
      }
    });
  });

  // Pass cellMeta to Handsontable via props to update the table
  if (hot) {
    hot.updateSettings({
      cells: function(row, col) {
        if (cellMeta[row] && cellMeta[row][col]) {
          return cellMeta[row][col];
        }
        return {}; // No additional settings for normal cells
      }
    });
  }
}




async function fetchApiData(classType) {
  console.log("Fetching API data for:", classType);

  // If the class type hasn't changed, avoid re-fetching
  if (classType === previousClassValue) {
    console.log("Class hasn't changed, skipping fetch...");
    return;
  }

  previousClassValue = classType;

  let apiUrls = [];
  columnHeaders = [];  // Clear previous column headers
  tableData = [];      // Clear previous table data

  // Set URLs for the selected class type
  if (classType === 'equity') {
    apiUrls = ["https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency=JPY&apikey=demo"];
  } else if (classType === 'bonds') {
    apiUrls = ["https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=BTC&to_currency=EUR&apikey=demo"];
  } else if (classType === 'daily') {
    apiUrls = [
      "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=msft&apikey=Rqvl",
      "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=aapl&apikey=Rqvl"
    ];
  } else if (classType === 'earnings') {
    apiUrls = ["https://www.alphavantage.co/query?function=EARNINGS&symbol=IBM&apikey=demo"];
  }

  if (apiUrls.length === 0) {
    console.log("No API URLs found for class type:", classType);
    return;
  }

  try {
    console.log("Fetching data for: ${classType}");
    let allData = [];

    if (classType === 'earnings') {
      let response = await fetch(apiUrls[0]);
      if (!response.ok) throw new Error('Failed to fetch data');

      const rawData = await response.json();
      console.log("Raw earnings data:", rawData);

      const earnings = rawData.annualEarnings.slice(0, 5);
      columnHeaders = earnings.map(e => formatDate(e.fiscalDateEnding));
      tableData = [earnings.map(e => e.reportedEPS)];

      if (showTicker) {
        columnHeaders.unshift("Ticker");
        tableData.forEach(row => row.unshift("IBM"));
      }
    } else {
      // Fetch data for other classes
      for (let url of apiUrls) {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch data');

        const rawData = await response.json();
        console.log("Raw data for ${classType}:", rawData);

        const firstKey = Object.keys(rawData)[0];
        const apiData = rawData[firstKey];

        if (apiData) {
          if (columnHeaders.length === 0) {
            columnHeaders = Object.keys(apiData);
          }
          allData.push(Object.values(apiData));
        }
      }
      tableData = allData;
    }

    console.log("Updated columnHeaders:", columnHeaders);
    console.log("Updated tableData:", tableData);
  } catch (err) {
    error = err.message;
    console.error('API fetch error:', error);
  }
}



  function formatDate(dateString) {
    const date = new Date(dateString);
    return date.toLocaleString('en-US', { month: 'short', year: 'numeric' });
  }

  function handleClassChange() {
  console.log("Changing class to:", classSelectValue);

  // Clear previous data to reset the table state
  columnHeaders = [];
  tableData = [];
  error = null;  // Clear any previous errors

  // Reset previousClassValue when switching to Bitcoin
  if (classSelectValue === 'bitcoin') {
    previousClassValue = null;  // Force the re-fetch of data for Bitcoin
    console.log("Fetching Bitcoin data...");
    fetchBitcoinData();  // Fetch Bitcoin data if selected
  } else if (classSelectValue === 'equity') {
    console.log("Fetching Equity data...");
    fetchApiData('equity');  // Fetch Equity data if selected
  } else {
    console.log("Fetching data for ${classSelectValue}...");
    fetchApiData(classSelectValue);  // Fetch other class data (bonds, daily, etc.)
  }
}



  function toggleTicker() {
    if (classSelectValue === "earnings") {
      previousClassValue = null;
      fetchApiData(classSelectValue);
    }
  }



  onMount(() => {
    fetchApiData(classSelectValue);
  });
</script>

<main>
  <header class="header">
    <h1>My Dashboard</h1>
  </header>
  <Handsontable {tableData} {columnHeaders} {highlightedCells} />
  <div class="layout">
    <div class="left-panel">
      <select id="classSelect" bind:value={classSelectValue} on:change={handleClassChange}>
        <option value="equity">Equity</option>
        <option value="bonds">Bonds</option>
        <option value="daily">Daily</option>
        <option value="earnings">Earnings</option>
        <option value="bitcoin">Bitcoin</option> <!-- Add Bitcoin option -->
      </select>

      {#if classSelectValue === 'bonds'}
        <!-- Country Dropdown -->
        <div class="custom-dropdown">
          <div class="dropdown-toggle" on:click={toggleCountryDropdown}>
            {#if selectedCountry}
              <img src={"https://flagcdn.com/w40/" + selectedCountry.code + ".png"} alt={selectedCountry.name} class="flag" />
              {selectedCountry.name}
            {:else}
              Select a country
            {/if}
          </div>

          {#if countryDropdownOpen}
            <div class="dropdown-menu">
              {#each countries as country}
                <div class="dropdown-item" on:click={() => selectCountry(country)}>
                  <img src={"https://flagcdn.com/w40/" + country.code + ".png"} alt={country.name} class="flag" />
                  {country.name}
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}

      {#if classSelectValue === "earnings"}
        <label>
          <input type="checkbox" bind:checked={showTicker} on:change={toggleTicker} />
          Show Ticker
        </label>
      {/if}
    </div>

    <div class="table-panel">
      <Handsontable {tableData} {columnHeaders} />
    </div>

    <div class="right-panel">
      <h2>Additional Information</h2>
      <div class="toggle-container">
        <button class="toggle-button" class:active={!isAfter} on:click={toggle}>Left</button>
        <button class="toggle-button" class:active={isAfter} on:click={toggle}>Right</button>
      </div>
    </div>
  </div>
</main>


<style>
  body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .header {
    background-color: #4CAF50;
    padding: 10px 20px;
    display: flex;
    align-items: center;
    color: white;
    border-bottom: 2px solid #ccc;
  }

  .logo {
    width: 40px;
    margin-right: 20px;
  }

  .header h1 {
    font-size: 24px;
    margin: 0;
  }

  .layout {
    display: flex;
    justify-content: space-between;
    gap: 20px;
    padding: 20px;
    height: calc(100vh - 80px);
  }

  .left-panel, .right-panel {
    flex: 1;
    max-width: 250px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
  }

  .table-panel {
    flex: 2;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #fff;
    min-height: 300px;
    overflow: hidden;
  }

  .button-container {
    margin-top: 20px;
    display: flex;
    justify-content: flex-start;
  }

  button {
    background-color: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    font-size: 16px;
    border-radius: 5px;
    cursor: pointer;
  }

  .custom-dropdown {
    position: relative;
    margin-top: 10px;
  }

  .dropdown-toggle {
    border: 1px solid #ccc;
    padding: 5px 10px;
    cursor: pointer;
    background: white;
    display: flex;
    align-items: center;
  }

  .dropdown-menu {
    position: absolute;
    top: 100%;
    left: 0;
    background: white;
    border: 1px solid #ccc;
    width: 100%;
    z-index: 10;
  }

  .dropdown-item {
    padding: 5px;
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  .dropdown-item:hover {
    background: #f0f0f0;
  }

  .flag {
    width: 20px;
    height: 15px;
    margin-right: 10px;
  }

  .toggle-container {
    display: flex;
    gap: 10px;
    margin-top: 20px;
  }

  .toggle-button {
    padding: 8px 16px;
    border: none;
    cursor: pointer;
    background-color: #ddd;
    border-radius: 5px;
    transition: 0.3s;
  }

  .toggle-button.active {
    background-color: #4CAF50;
    color: white;
  }
  .highlight {
    background-color: yellow;  /* You can change this to any color you want */
    color: red; /* Change the text color for better contrast */
    font-weight: bold; /* Optional: make the highlighted text bold */
  }

  .highlight {
    background-color: violet !important; /* Highlighting color */
  }

  .layout {
    display: flex;
  }

  .left-panel {
    width: 20%;
    padding: 10px;
  }

  .right-panel {
    width: 20%;
    padding: 10px;
  }

  .table-panel {
    flex-grow: 1;
    padding: 10px;
  }

  .toggle-container {
    display: flex;
    justify-content: space-between;
  }

  .toggle-button.active {
    background-color: lightblue;
  }

  .custom-dropdown .dropdown-menu {
    position: absolute;
    background-color: white;
    border: 1px solid #ccc;
    width: 200px;
    z-index: 10;
  }
</style>

