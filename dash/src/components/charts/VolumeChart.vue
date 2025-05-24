<script setup>
import { defineProps, computed } from 'vue';
import { Bar } from 'vue-chartjs';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js';

ChartJS.register(
  CategoryScale,
  LinearScale,
  BarElement,
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
      label: 'Volumen',
      data: props.data.map(d => d.volume),
      backgroundColor: '#2ecc71',
      borderColor: '#2ecc71',
      borderWidth: 1,
      borderRadius: 4
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
      text: 'Volumen de Trading',
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
        text: 'Volumen'
      }
    }
  }
};
</script>

<template>
  <div class="chart-container">
    <Bar :data="chartData" :options="chartOptions" />
  </div>
</template>

<style scoped>
.chart-container {
  height: 400px;
}
</style>
