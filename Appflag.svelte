<script>
  import Handsontable from './Handsontable.svelte';
  import { onMount, afterUpdate } from 'svelte';
  import noUiSlider from 'nouislider';
  import 'nouislider/dist/nouislider.css';

  let classSelectValue = 'equity';
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

  function initializeSlider() {
    if (sliderContainer && !sliderInstance) {
      sliderInstance = noUiSlider.create(sliderContainer, {
        start: rangeValues,
        connect: true,
        range: {
          min: 2020,
          max: 2044
        },
        step: 1,
        tooltips: true,
        format: {
          to: value => Math.round(value),
          from: value => Number(value)
        }
      });

      sliderInstance.on('update', values => {
        rangeValues = values.map(value => Math.round(value));
      });
    }
  }

  function handleClassChange() {
    if (classSelectValue === 'bonds') {
      setTimeout(() => {
        initializeSlider();
      }, 10);
    } else {
      if (sliderInstance) {
        sliderInstance.destroy();
        sliderInstance = null;
      }
    }
  }

  function saveChanges() {
    if (savedChanges.length === 0) {
      alert("No changes to save!");
      return;
    }

    let textContent = savedChanges
      .map(({ row, column, oldValue, newValue }) => 
        `Row: ${row}, Column: ${column}, Old: ${oldValue}, New: ${newValue}`
      )
      .join("\n");

    const blob = new Blob([textContent], { type: "text/plain" });
    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = url;
    a.download = "saved_changes.txt";
    document.body.appendChild(a);
    a.click();

    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function handleTableUpdate(event) {
    savedChanges = event.detail.savedChanges;
    console.log("Updated changes received:", savedChanges);
  }

  onMount(() => {
    handleClassChange();
  });

  afterUpdate(() => {
    handleClassChange();
  });
</script>

<main>
  <header class="header">
    <img src="/path/to/your/logo.png" alt="Logo" class="logo" />
    <h1>My Dashboard</h1>
  </header>

  <div class="layout">
    <div class="left-panel">
      <h2>Options</h2>
      <label for="classSelect">Class:</label>
      <select id="classSelect" bind:value={classSelectValue} on:change={handleClassChange}>
        <option value="equity">Equity</option>
        <option value="bonds">Bonds</option>
      </select>
      <br />

      {#if classSelectValue === 'bonds'}
        <div class="slider-container">
          <div class="input-title">Select Range</div>
          <div class="spacer"></div>
          <div bind:this={sliderContainer}></div>
        </div>

        <!-- Custom Dropdown -->
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
      <Handsontable {rangeValues} on:updateChanges={handleTableUpdate} />
    </div>

    <div class="right-panel">
      <h2>Additional Information</h2>
      <p>Some content or charts can go here!</p>
      <div class="button-container">
        <button on:click={saveChanges}>Save Changes</button>
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
</style>
