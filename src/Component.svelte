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

  export let averageLabel;

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

  function handleDataWithFilter(rows) {
    if (rows) {
      console.log(rows);

      // Use the most recently created row
      const filteredRow = rows.reduce(
        (latest, row) =>
          new Date(row.createdAt) > new Date(latest.createdAt) ? row : latest,
        rows[0]
      );

      console.log(filteredRow);

      options.labels = [];
      options.series = [];

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
      values.push(filteredRow[valueField1]);
      values.push(filteredRow[valueField2]);
      values.push(filteredRow[valueField3]);
      values.push(filteredRow[valueField4]);
      values.push(filteredRow[valueField5]);
      values.push(filteredRow[valueField6]);
      values.push(filteredRow[valueField7]);
      values.push(filteredRow[valueField8]);

      options.series.push({ name: filteredRow[serieField], data: values });

      // CALCULATE AVERAGE
      // Initialize an object for storing sums
      const sum = {};
      const count = {};
      const fields = [
        valueField1,
        valueField2,
        valueField3,
        valueField4,
        valueField5,
        valueField6,
        valueField7,
        valueField8,
      ];

      // Initialize sum and count for each field
      fields.forEach((field) => {
        sum[field] = 0;
        count[field] = 0;
      });

      // Sum the values for the hardcoded fields
      rows.forEach((row) => {
        fields.forEach((field) => {
          if (field in row) {
            sum[field] += row[field];
            count[field] += 1;
          }
        });
      });

      // Compute the average for each field
      let averageObject = { nom: averageLabel };
      fields.forEach((field) => {
        averageObject[field] = sum[field] / count[field];
      });

      console.log(options.labels);

      let averageValues = [];
      averageValues.push(averageObject[valueField1]);
      averageValues.push(averageObject[valueField2]);
      averageValues.push(averageObject[valueField3]);
      averageValues.push(averageObject[valueField4]);
      averageValues.push(averageObject[valueField5]);
      averageValues.push(averageObject[valueField6]);
      averageValues.push(averageObject[valueField7]);
      averageValues.push(averageObject[valueField8]);

      options.series.push({
        name: averageObject[serieField],
        data: averageValues,
      });
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
    // handleData(dataProvider?.rows);
    handleDataWithFilter(dataProvider?.rows);
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
