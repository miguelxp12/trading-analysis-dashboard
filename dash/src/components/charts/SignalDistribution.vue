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

const chartData = computed(() => {
  const signalCounts = props.data.reduce((acc, curr) => {
    const label = curr.label || 'HOLD';
    acc[label] = (acc[label] || 0) + 1;
    return acc;
  }, {});

  const labels = Object.keys(signalCounts);
  const data = Object.values(signalCounts);
  const total = data.reduce((a, b) => a + b, 0);

  return {
    labels,
    datasets: [
      {
        label: 'Señales',
        data,
        backgroundColor: labels.map(label => {
          switch (label) {
            case 'BUY': return '#2ecc71';
            case 'SELL': return '#e74c3c';
            default: return '#3498db';
          }
        })
      }
    ]
  };
});

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top'
    },
    title: {
      display: true,
      text: 'Distribución de Señales de Trading',
      font: {
        size: 16,
        weight: 'bold'
      }
    },
    tooltip: {
      callbacks: {
        label: (context) => {
          const total = context.dataset.data.reduce((a, b) => a + b, 0);
          const percentage = ((context.raw / total) * 100).toFixed(1);
          return `${context.label}: ${context.raw} (${percentage}%)`;
        }
      }
    }
  },
  scales: {
    y: {
      title: {
        display: true,
        text: 'Cantidad de Señales'
      },
      beginAtZero: true
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