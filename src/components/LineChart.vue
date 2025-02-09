<template>
    <div>
      <canvas ref="chartCanvas"></canvas>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, watch } from 'vue';
  import { Chart, registerables } from 'chart.js';
  
  Chart.register(...registerables);
  
  const props = defineProps(['chartData']);
  const chartCanvas = ref(null);
  let chartInstance = null;
  
  const createChart = () => {
    if (chartInstance) {
      chartInstance.destroy();
    }
  
    chartInstance = new Chart(chartCanvas.value, {
      type: 'line',
      data: {
        labels: props.chartData.labels,
        datasets: [
          {
            label: 'DE Price',
            data: props.chartData.DE,
            borderColor: 'red',
            fill: false
          },
          {
            label: 'GR Price',
            data: props.chartData.GR,
            borderColor: 'blue',
            fill: false
          },
          {
            label: 'FR Price',
            data: props.chartData.FR,
            borderColor: 'green',
            fill: false
          }
        ]
      },
      options: {
        responsive: true,
        scales: {
          x: { title: { display: true, text: 'Time' } },
          y: { title: { display: true, text: 'Price' } }
        }
      }
    });
  };
  
  // Watch for data changes and update chart
  watch(() => props.chartData, createChart, { deep: true });
  
  onMounted(createChart);
  </script>
  