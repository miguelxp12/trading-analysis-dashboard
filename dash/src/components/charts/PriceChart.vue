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

const chartData = computed(() => {
  if (!props.data || props.data.length === 0) {
    return { labels: [], datasets: [] }; // Return empty structure if no data
  }
  return {
    labels: props.data.map(d => {
      const date = new Date(d.timestamp);
      // Using a more concise date/time format for x-axis labels
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', day: '2-digit', month: 'short' });
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
      text: 'Precios de PEPE/USDT (últimas 100 velas)', // Title is descriptive
      font: {
        size: 16,
        weight: 'bold'
      }
    },
    tooltip: {
      callbacks: {
        title: function(tooltipItems) {
          // Display a more readable timestamp in the tooltip title
          const date = new Date(props.data[tooltipItems[0].dataIndex].timestamp);
          return date.toLocaleString([], { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit' });
        },
        label: function(context) {
          let label = context.dataset.label || '';
          if (label) {
            label += ': ';
          }
          if (context.parsed.y !== null) {
            label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USDT' }).format(context.parsed.y);
          }
          return label;
        }
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
    <div v-if="!props.data || props.data.length === 0" class="loading-indicator">
      Cargando datos del gráfico...
    </div>
    <Line v-else :data="chartData" :options="chartOptions" />
  </div>
</template>

<style scoped>
.loading-indicator {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  font-style: italic;
  color: #888;
}
.chart-container {
  height: 400px;
}
</style>