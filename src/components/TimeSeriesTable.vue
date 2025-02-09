<template>
  <div>
    <h1>Time Series Data</h1>

    <!-- Date Range Picker -->
    <DateRangePicker @filter="filterData" />

    <!-- Checkbox for showing/hiding timeseries -->
    <div class="checkbox-group">
      <label><input type="checkbox" v-model="showDE" /> Show DE Price</label>
      <label><input type="checkbox" v-model="showGR" /> Show GR Price</label>
      <label><input type="checkbox" v-model="showFR" /> Show FR Price</label>
    </div>

    <!-- Table to display timeseries data -->
    <table>
      <thead>
        <tr>
          <th>Timestamp</th>
          <th v-if="showDE">DE Price</th>
          <th v-if="showGR">GR Price</th>
          <th v-if="showFR">FR Price</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(entry, index) in filteredData" :key="index">
          <td>{{ entry.DateTime.format('DD-MM-YYYY HH:mm') }}</td>
          <td v-if="showDE">
            <input 
              type="number" 
              v-model="entry.ENTSOE_DE_DAM_Price" 
              :class="{'invalid': !isValid(entry.ENTSOE_DE_DAM_Price)}"
              @input="updateChartData"
            />
          </td>
          <td v-if="showGR">
            <input 
              type="number" 
              v-model="entry.ENTSOE_GR_DAM_Price" 
              :class="{'invalid': !isValid(entry.ENTSOE_GR_DAM_Price)}"
              @input="updateChartData"
            />
          </td>
          <td v-if="showFR">
            <input 
              type="number" 
              v-model="entry.ENTSOE_FR_DAM_Price" 
              :class="{'invalid': !isValid(entry.ENTSOE_FR_DAM_Price)}"
              @input="updateChartData"
            />
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Validation Message -->
    <div v-if="validationMessage" class="validation-message">
      {{ validationMessage }}
    </div>

    <!-- Chart Component -->
    <LineChart :chartData="chartData" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import dayjs from 'dayjs';
import isBetween from 'dayjs/plugin/isBetween';
import jsonData from '../assets/timeseries.json';
import DateRangePicker from '../components/DateRangePicker.vue';
import LineChart from '../components/LineChart.vue';

// Extend dayjs with isBetween
dayjs.extend(isBetween);

const data = ref([]);
const filteredData = ref([]);
const validationMessage = ref("");

const showDE = ref(true);
const showGR = ref(true);
const showFR = ref(true);

onMounted(() => {
  data.value = jsonData.map(item => ({
    DateTime: dayjs(item.DateTime), // Store as a `dayjs` object
    ENTSOE_DE_DAM_Price: item.ENTSOE_DE_DAM_Price,
    ENTSOE_GR_DAM_Price: item.ENTSOE_GR_DAM_Price,
    ENTSOE_FR_DAM_Price: item.ENTSOE_FR_DAM_Price
  }));
  filteredData.value = [...data.value];
  updateChartData();
});

// Function to validate input
const isValid = (value) => !isNaN(value) && value >= -2000 && value <= 2000;

// Function to validate the input when user finishes editing
const validateInput = (entry, country) => {
  const valueKey = `ENTSOE_${country}_DAM_Price`;
  const value = entry[valueKey];

  if (!isValid(value)) {
    validationMessage.value = `Invalid value for ${country} Price. Please enter a number between -2000 and 2000.`;
  } else {
    validationMessage.value = "";
  }
};

// Function to filter data based on date range
const filterData = (dates) => {
  const { startDate, endDate } = dates;

  if (startDate && endDate) {
    const start = dayjs(startDate, 'DD-MM-YYYY').startOf('day');
    const end = dayjs(endDate, 'DD-MM-YYYY').endOf('day');

    filteredData.value = data.value.filter(entry => 
      entry.DateTime.isBetween(start, end, null, '[]')
    );
  } else {
    filteredData.value = [...data.value];
  }
  updateChartData();
};

// Compute chart data dynamically
const chartData = ref({
  labels: [],
  DE: [],
  GR: [],
  FR: []
});

// Function to update the chart when table values change
const updateChartData = () => {
  chartData.value = {
    labels: filteredData.value.map(entry => entry.DateTime.format('DD-MM-YYYY HH:mm')),
    DE: showDE.value ? filteredData.value.map(entry => entry.ENTSOE_DE_DAM_Price) : [],
    GR: showGR.value ? filteredData.value.map(entry => entry.ENTSOE_GR_DAM_Price) : [],
    FR: showFR.value ? filteredData.value.map(entry => entry.ENTSOE_FR_DAM_Price) : []
  };
};
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}
th, td {
  border: 1px solid black;
  padding: 8px;
  text-align: left;
}
th {
  background-color: #121212;
}
input {
  width: 100%;
  padding: 5px;
  border: 1px solid #ccc;
}
input.invalid {
  border-color: red;
}
.validation-message {
  color: red;
  font-weight: bold;
}
.checkbox-group {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
}
</style>
