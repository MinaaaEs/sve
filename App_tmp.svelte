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

  function toggle() {
    isAfter = !isAfter;
  }

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

  async function fetchBitcoinData() {
    try {
      let response = await fetch("https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=EUR&apikey=demo");
      if (!response.ok) throw new Error("Failed to fetch Bitcoin data");
      let data = await response.json();
      let timeSeries = data["Time Series (Digital Currency Daily)"];
      if (!timeSeries) throw new Error("Invalid API response format");
      
      let dates = Object.keys(timeSeries).sort((a, b) => new Date(b) - new Date(a));
      let selectedDate = isAfter ? dates[1] : dates[0]; // Toggle-based selection
      let selectedData = timeSeries[selectedDate];
      
      columnHeaders = ["Date", "Open", "High", "Low", "Close", "Volume"];
      tableData = [[selectedDate, selectedData["1. open"], selectedData["2. high"], selectedData["3. low"], selectedData["4. close"], selectedData["5. volume"]]];
    } catch (err) {
      error = err.message;
      console.error("Bitcoin API fetch error:", error);
    }
  }

  async function fetchApiData(classType) {
    if (classType !== "earnings" && classType === previousClassValue) return;
    previousClassValue = classType;

    let apiUrls = [];
    columnHeaders = [];
    tableData = [];

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


    if (apiUrls.length === 0) return;

    try {
      console.log(`Fetching data for: ${classType}`);
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
        for (let url of apiUrls) {
          const response = await fetch(url);
          if (!response.ok) throw new Error('Failed to fetch data');

          const rawData = await response.json();
          console.log(`Raw data for ${classType}:`, rawData);

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
    fetchApiData(classSelectValue);
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

  <div class="layout">
    <div class="left-panel">
      <label for="classSelect">Class:</label>
      <select id="classSelect" bind:value={classSelectValue} on:change={handleClassChange}>
        <option value="equity">Equity</option>
        <option value="bonds">Bonds</option>
        <option value="daily">Daily</option>
        <option value="earnings">Earnings</option>
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
        <button class="toggle-button" class:active={!isAfter} on:click={() => isAfter = false}>Left</button>
        <button class="toggle-button" class:active={isAfter} on:click={() => isAfter = true}>Right</button>
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
</style>

