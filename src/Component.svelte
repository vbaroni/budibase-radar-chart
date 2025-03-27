<script>
  import { getContext } from "svelte";
  import { chart } from "svelte-apexcharts";
  import { merge } from "lodash";

  export let dataProvider; //
  export let serieField; //
  export let valueField1; //
  export let labelField1; //
  export let valueField2; //
  export let labelField2; //
  export let valueField3; //
  export let labelField3; //
  export let valueField4; //
  export let labelField4; //
  export let valueField5; //
  export let labelField5; //
  export let valueField6; //
  export let labelField6; //
  export let valueField7; //
  export let labelField7; //
  export let valueField8; //
  export let labelField8; //
  export let chartHeight; //
  export let chartWidth; //
  export let chartTitle; //
  export let minAxis; //
  export let maxAxis; //
  export let axisTickAmount; //
  export let colorPalette; //
  export let displayDataLabels; //
  export let advancedChartOptions; //

  const loading = getContext("loading");
  let canBeDisplayed = true;
  let errorMessages = [];

  // ----------------------------------------------------------------
  // Manage all options of the chart
  var options = {};
  options.chart = {
    height: chartHeight,
    type: "radar",
    toolbar: { show: false },
  };
  if (chartWidth != null) {
    options.chart.width = chartWidth;
  }
  if (chartTitle != null) {
    options.title = { text: chartTitle };
  }
  options.yaxis = {};
  if (minAxis != null) {
    options.yaxis.min = minAxis;
  }
  if (maxAxis != null) {
    options.yaxis.max = maxAxis;
  }
  if (axisTickAmount != null) {
    options.yaxis.tickAmount = axisTickAmount;
  }
  options.dataLabels = {
    enabled: displayDataLabels,
    background: { enabled: true, borderRadius: 2 },
  };
  options.theme = { palette: colorPalette };
  options.plotOptions = {
    radar: {
      polygons: {
        strokeColor: "#e8e8e8",
        fill: {
          colors: ["#f8f8f8", "#fff"],
        },
      },
    },
  };

  // Merge advanced chart options with options computed just before
  if (advancedChartOptions != null) {
    options = merge(options, advancedChartOptions);
  }

  // ----------------------------------------------------------------
  // Manage data of the chart
  options.series = [];

  function nvl(labelName, fieldName) {
    return labelName == null ? fieldName : labelName;
  }

  // Handle data when rows retrieved
  function handleData(rows) {
    if (rows) {
      options.labels = [];
      options.series = [];

      for (const element of rows) {
        // Manage labels
        options.labels.push(nvl(labelField1, valueField1));
        options.labels.push(nvl(labelField2, valueField2));
        options.labels.push(nvl(labelField3, valueField3));
        options.labels.push(nvl(labelField4, valueField4));
        options.labels.push(nvl(labelField5, valueField5));
        options.labels.push(nvl(labelField6, valueField6));
        options.labels.push(nvl(labelField7, valueField7));
        options.labels.push(nvl(labelField8, valueField8));

        // Manage values
        let values = [];
        values.push(element[valueField1]);
        values.push(element[valueField2]);
        values.push(element[valueField3]);
        values.push(element[valueField4]);
        values.push(element[valueField5]);
        values.push(element[valueField6]);
        values.push(element[valueField7]);
        values.push(element[valueField8]);

        options.series.push({ name: element[serieField], data: values });
      }
    } else {
      canBeDisplayed = false;
      errorMessages.push("No data");
    }
  }

  // Load data and manage them
  $: rows = $loading
    ? new Array(dataProvider?.limit > 16 ? 16 : dataProvider?.limit).fill({})
    : dataProvider?.rows;
  $: if (dataProvider?.rows && dataProvider?.rows.length > 0) {
    handleData(dataProvider?.rows);
  }
</script>

{#if canBeDisplayed}
  <div use:chart={options} />
{:else}
  <!-- Display error message of bad configuration -->
  <div class="spectrum-InLineAlert spectrum-InLineAlert--notice">
    <div class="spectrum-InLineAlert-header">
      Bubble chart configuration error
      <svg
        class="spectrum-Icon spectrum-Icon--sizeM spectrum-InLineAlert-icon"
        focusable="false"
        aria-hidden="true"
      >
        <use xlink:href="#spectrum-icon-18-Alert" />
      </svg>
    </div>
    <div class="spectrum-InLineAlert-content">
      <ul>
        {#each errorMessages as message}
          <li>{message}</li>
        {/each}
      </ul>
    </div>
  </div>
{/if}
