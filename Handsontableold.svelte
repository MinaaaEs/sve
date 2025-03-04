<script>
  import { onMount } from 'svelte';
  import Handsontable from 'handsontable';
  import 'handsontable/dist/handsontable.full.css';

  let container;

  onMount(() => {
    const data = [
      ['Year', 'Q1', 'Q2', 'Q3', 'Q4'],
      ['2020', 120, 130, 140, 150],
      ['2021', 110, 120, 130, 140],
      ['2022', 115, 125, 135, 145]
    ];

    new Handsontable(container, {
      data: data,
      colHeaders: true,
      rowHeaders: true,
      width: '100%',
      height: '300px',
      licenseKey: 'non-commercial-and-evaluation'
    });
  });
</script>

<style>
  div {
    margin: 20px;
  }
</style>

<div bind:this={container}></div>
