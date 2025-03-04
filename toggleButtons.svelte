<script>
  let isToggled = false;

  // Function to toggle the state
  function toggle() {
    isToggled = !isToggled;
  }
</script>

<div>
  <!-- Button 1 -->
  <button on:click={toggle} class={isToggled ? 'active' : ''}>
    {isToggled ? 'Deactivate' : 'Activate'}
  </button>

  <!-- Button 2 (will be disabled when the state is active) -->
  <button on:click={toggle} class={!isToggled ? 'active' : ''} disabled={isToggled}>
    {isToggled ? 'Active' : 'Inactive'}
  </button>
</div>

<style>
  button {
    padding: 10px 20px;
    font-size: 16px;
    margin: 10px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button.active {
    background-color: #4caf50; /* Green when active */
    color: white;
  }

  button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
</style>
