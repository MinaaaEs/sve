<script>
  import Handsontable from './Handsontable.svelte';
  import { onMount, afterUpdate } from 'svelte';
  import noUiSlider from 'nouislider';
  import 'nouislider/dist/nouislider.css';

  let classSelectValue = 'equity';
  let sliderContainer;
  let sliderInstance;
  let rangeValues = [2025, 2040];

  let savedChanges = []; // Stores the saved cell changes

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
    console.log("Saved Changes:", savedChanges);
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
  <!-- Header Section -->
  <header class="header">
    <img src="/path/to/your/logo.png" alt="Logo" class="logo" />
    <h1>My Dashboard</h1>
  </header>

  <!-- Layout Section -->
  <div class="layout">
    <!-- Left Panel -->
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
      {/if}
    </div>

    <!-- Table Panel -->
    <div class="table-panel">
      <Handsontable {rangeValues} on:updateChanges={handleTableUpdate} />
    </div>

    <!-- Right Panel -->
    <div class="right-panel">
      <h2>Additional Information</h2>
      <p>Some content or charts can go here!</p>
    </div>
  </div>

  <!-- Save Changes Button -->
  <div class="button-container">
    <button on:click={saveChanges}>Save Changes</button>
  </div>
</main>

<style>
  /* Ensure layout remains intact */
  body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  /* Header Styling */
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

  /* Layout */
  .layout {
    display: flex;
    justify-content: space-between;
    gap: 20px;
    padding: 20px;
    height: calc(100vh - 80px); /* Ensure it doesn't collapse */
  }

  /* Left Panel */
  .left-panel {
    flex: 1;
    max-width: 250px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
  }

  /* Table Panel */
  .table-panel {
    flex: 2;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #fff;
    min-height: 300px; /* Prevents collapsing */
    overflow: hidden;
  }

  /* Right Panel */
  .right-panel {
    flex: 1;
    max-width: 250px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
  }

  /* Save Button */
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

  button:hover {
    background-color: #45a049;
  }
</style>



