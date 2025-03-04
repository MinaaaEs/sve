<script>
  import Handsontable from './Handsontable.svelte';
  import { onMount } from 'svelte';

  let classSelectValue = 'equity'; // Default to equity
  let tableData = [];
  let columnHeaders = [];
  let error = null;
  let sliderContainer;
  let sliderInstance;
  let rangeValues = [2025, 2040];
  let savedChanges = []; 
  let selectedCountry = null;
  let dropdownOpen = false;

  const countries = [
    { name: "Germany", code: "de" },
    { name: "France", code: "fr" },
    { name: "Spain", code: "es" },
    { name: "Italy", code: "it" },
    { name: "Netherlands", code: "nl" }
  ];

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }

  function selectCountry(country) {
    selectedCountry = country;
    dropdownOpen = false;
  }

  let previousClassValue = null;

  async function fetchApiData(classType) {
    if (classType === previousClassValue) return;
    previousClassValue = classType;

    let apiUrls = [];

    if (classType === 'equity') {
      apiUrls = ["https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency=JPY&apikey=demo"];
    } else if (classType === 'bonds') {
      apiUrls = ["https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=BTC&to_currency=EUR&apikey=demo"];
    } else if (classType === 'daily') {
      apiUrls = [
        "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=msft&apikey=Rqvl",
        "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=aapl&apikey=Rqvl"
      ];
    }

    if (apiUrls.length === 0) {
      tableData = [];
      columnHeaders = [];
      console.log("No API call for this selection, table cleared.");
      return;
    }

    try {
      console.log(`Fetching data for: ${classType}`);
      let allData = [];
      
      for (let url of apiUrls) {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch data');

        const rawData = await response.json();
        const firstel = Object.keys(rawData)[0];
        console.log(`Raw data for ${classType}:`, rawData);

        const apiData = rawData[firstel];

        if (apiData) {
          console.log(`apiData structure for ${classType}:`, apiData);
          if (columnHeaders.length === 0) {
            columnHeaders = Object.keys(apiData);
          }
          allData.push(Object.values(apiData));
        }
      }

      tableData = allData;
      console.log(`Updated columnHeaders for ${classType}:`, columnHeaders);
      console.log(`Updated tableData for ${classType}:`, tableData);
    } catch (err) {
      error = err.message;
      console.error('API fetch error:', error);
    }
  }

  function handleClassChange() {
    fetchApiData(classSelectValue);
  }

  onMount(() => {
    fetchApiData(classSelectValue);
  });
</script>

<main>
  <header class="header">
    <img src="/path/to/your/logo.png" alt="Logo" class="logo" />
    <h1>My Dashboard</h1>
  </header>

  <div class="layout">
    <div class="left-panel">
      <label for="classSelect">Class:</label>
      <select id="classSelect" bind:value={classSelectValue} on:change={handleClassChange}>
        <option value="equity">Equity</option>
        <option value="bonds">Bonds</option>
        <option value="daily">Daily</option>
      </select>
      <br />
      {#if classSelectValue === 'bonds'}
        <div class="slider-container">
          <div class="input-title">Select Range</div>
          <div class="spacer"></div>
        </div>

        <div class="custom-dropdown">
          <div class="dropdown-toggle" on:click={toggleDropdown}>
            {#if selectedCountry}
              <img src={`https://flagcdn.com/w40/${selectedCountry.code}.png`} alt={selectedCountry.name} class="flag" />
              {selectedCountry.name}
            {:else}
              Select a country
            {/if}
          </div>

          {#if dropdownOpen}
            <div class="dropdown-menu">
              {#each countries as country}
                <div class="dropdown-item" on:click={() => selectCountry(country)}>
                  <img src={`https://flagcdn.com/w40/${country.code}.png`} alt={country.name} class="flag" />
                  {country.name}
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}
    </div>

    <div class="table-panel">
      <Handsontable {tableData} {columnHeaders} />
    </div>

    <div class="right-panel">
      <h2>Additional Information</h2>
      <p>Some content or charts can go here!</p>
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
</style>
