<script setup>
import { defineProps, computed } from 'vue';
import { Line } from 'vue-chartjs';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js';

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
);

const props = defineProps({
  data: {
    type: Array,
    required: true
  }
});

const chartData = computed(() => ({
  labels: props.data.map(d => {
    const date = new Date(d.timestamp);
    return date.toLocaleString();
  }),
  datasets: [
    {
      label: 'Precio de cierre',
      data: props.data.map(d => d.close),
      borderColor: '#1f77b4',
      backgroundColor: '#1f77b4',
      tension: 0.1
    }
  ]
}));

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top'
    },
    title: {
      display: true,
      text: 'Precios de PEPE/USDT (últimas 100 velas)',
      font: {
        size: 16,
        weight: 'bold'
      }
    }
  },
  scales: {
    x: {
      ticks: {
        maxRotation: 45,
        minRotation: 45
      }
    },
    y: {
      title: {
        display: true,
        text: 'Precio en USDT'
      }
    }
  }
};
</script>

<template>
  <div class="chart-container">
    <Line :data="chartData" :options="chartOptions" />
  </div>
</template>

<style scoped>
.chart-container {
  height: 400px;
}
</style>