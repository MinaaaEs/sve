<script>
  import { onMount } from 'svelte';
  let selectedClass = 'Bond'; // Initially set to 'Bond'
  
  let sliderContainer;

  // Initialize slider only when Bond is selected
  let initSlider = false;

  // Function to toggle slider visibility
  const toggleSlider = () => {
    initSlider = selectedClass === 'Bond';

    // Initialize slider when Bond is selected
    if (initSlider && !sliderContainer.querySelector("#year-slider").ionRangeSlider) {
      $(sliderContainer).find('#year-slider').ionRangeSlider({
        type: "double",
        min: 2020,
        max: 2044,
        from: 2020,
        to: 2044,
        grid: true
      });
    } else if (!initSlider) {
      // Destroy slider when switching to Equity
      $(sliderContainer).find('#year-slider').ionRangeSlider("destroy");
    }
  };

  // Call toggleSlider whenever selectedClass changes
  $: {
    toggleSlider();
  }
</script>

<style>
  .header {
    font-weight: bold;
  }

  /* Layout styles for the three-panel structure */
  .app-container {
    display: flex;
    justify-content: space-between;
    gap: 20px;
  }

  .panel {
    flex: 1;
    padding: 20px;
    border: 1px solid #ddd;
  }

  .class-selector {
    margin-bottom: 20px;
  }

  .slider-container {
    margin-top: 20px;
  }

  .equity-content {
    margin-top: 20px;
  }
</style>

<div class="app-container">
  <!-- Left Panel: Class Selector -->
  <div class="panel">
    <div class="class-selector">
      <label for="class-selector">Select Class: </label>
      <select bind:value={selectedClass} id="class-selector">
        <option value="Bond">Bonds</option>
        <option value="Equity">Equity</option>
      </select>
    </div>

    <!-- Slider Section, only visible when 'Bond' is selected -->
    {#if selectedClass === 'Bond'}
      <div bind:this={sliderContainer} class="slider-container">
        <div class="header">Select Range</div>
        <input type="text" id="year-slider" />
      </div>
    {/if}
  </div>

  <!-- Center Panel: Main Table -->
  <div class="panel">
    <h3>Main Table</h3>
    <!-- Render Handsontable or any other table here -->
    <table>
      <thead>
        <tr>
          <th>Year</th>
          <th>Value</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>2020</td>
          <td>100</td>
        </tr>
        <tr>
          <td>2021</td>
          <td>110</td>
        </tr>
        <tr>
          <td>2022</td>
          <td>120</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Right Panel: Content for Equity -->
  <div class="panel">
    {#if selectedClass === 'Equity'}
      <div class="equity-content">
        <h3>Equity Table</h3>
        <!-- Example of Equity Table -->
        <table>
          <thead>
            <tr>
              <th>Equity Column 1</th>
              <th>Equity Column 2</th>
              <th>Equity Column 3</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>Data 1</td>
              <td>Data 2</td>
              <td>Data 3</td>
            </tr>
          </tbody>
        </table>
      </div>
    {/if}
  </div>
</div>
