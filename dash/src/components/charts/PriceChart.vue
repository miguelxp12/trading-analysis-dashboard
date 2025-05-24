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
  Legend,
  TimeScale // Import TimeScale for the x-axis
} from 'chart.js';
import 'chartjs-adapter-date-fns'; // Import the date adapter

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  TimeScale // Register TimeScale
);

const props = defineProps({
  data: {
    type: Array,
    required: true
  },
  signals: { // New prop for signals
    type: Array,
    default: () => []
  }
});

const chartData = computed(() => {
  if (!props.data || props.data.length === 0) {
    return { labels: [], datasets: [] };
  }

  const priceLineDataset = {
    label: 'Precio de cierre',
    // For time series, data should be {x: Date, y: value}
    data: props.data.map(d => ({ x: d.timestamp, y: d.close })),
    borderColor: '#1f77b4',
    backgroundColor: '#1f77b4',
    tension: 0.1,
    type: 'line', // Explicitly line type
    order: 1 // Render price line behind signals
  };

  const buySignals = props.signals
    .filter(s => s.type === 'BUY')
    .map(s => ({ x: s.x, y: s.y })); // s.x is already a Date object

  const sellSignals = props.signals
    .filter(s => s.type === 'SELL')
    .map(s => ({ x: s.x, y: s.y })); // s.x is already a Date object

  const datasets = [priceLineDataset];

  if (buySignals.length > 0) {
    datasets.push({
      label: 'Buy Signal',
      data: buySignals,
      type: 'scatter',
      backgroundColor: '#2ecc71', // Green
      borderColor: '#27ae60', // Darker green for border
      borderWidth: 1,
      pointRadius: 7,
      pointStyle: 'triangle',
      rotation: 0, // Pointing upwards
      showLine: false,
      order: 0 // Render on top
    });
  }

  if (sellSignals.length > 0) {
    datasets.push({
      label: 'Sell Signal',
      data: sellSignals,
      type: 'scatter',
      backgroundColor: '#e74c3c', // Red
      borderColor: '#c0392b', // Darker red for border
      borderWidth: 1,
      pointRadius: 7,
      pointStyle: 'triangle',
      rotation: 180, // Pointing downwards
      showLine: false,
      order: 0 // Render on top
    });
  }

  return {
    // Labels are not needed when x-axis is 'time' and data is {x,y}
    datasets
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
      text: 'Precios de PEPE/USDT con Señales', // Updated title
      font: {
        size: 16,
        weight: 'bold'
      }
    },
    tooltip: {
      callbacks: {
        title: function(tooltipItems) {
          const date = new Date(tooltipItems[0].parsed.x);
          // Using toLocaleString for a comprehensive date-time representation
          return date.toLocaleString([], { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit' });
        },
        label: function(context) {
          let label = context.dataset.label || '';
          if (context.dataset.label === 'Buy Signal' || context.dataset.label === 'Sell Signal') {
            // For signal points, the label is already "Buy Signal" or "Sell Signal"
            if (context.parsed.y !== null) {
              label += ` at ${new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USDT' }).format(context.parsed.y)}`;
            }
          } else { // For the price line
            if (label) {
              label += ': ';
            }
            if (context.parsed.y !== null) {
              label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USDT' }).format(context.parsed.y);
            }
          }
          return label;
        }
      }
    }
  },
  scales: {
    x: {
      type: 'time', // Crucial for plotting Date objects correctly
      time: {
        tooltipFormat: 'MMM dd, yyyy HH:mm:ss', // Format for tooltips
        // unit: 'minute', // Removed to allow Chart.js to auto-determine the unit
         displayFormats: { // Define how different time units are displayed on the axis
            millisecond: 'HH:mm:ss.SSS',
            second: 'HH:mm:ss',
            minute: 'HH:mm', // Display format for minutes
            hour: 'HH:00',   // Display format for hours
            day: 'MMM dd',   // Display format for days
            week: 'MMM dd',
            month: 'MMM yyyy',
            quarter: 'MMM yyyy',
            year: 'yyyy',
        }
      },
      ticks: {
        maxRotation: 45,
        minRotation: 45,
        source: 'auto', // Let Chart.js automatically determine ticks
        autoSkip: true,
        maxTicksLimit: 20 // Adjust for density
      },
       title: {
        display: true,
        text: 'Fecha / Hora'
      }
    },
    y: {
      title: {
        display: true,
        text: 'Precio en USDT'
      },
      beginAtZero: false // Price charts usually don't start at zero
    }
  }
};
</script>

<template>
  <div class="chart-container">
    <div v-if="!props.data || props.data.length === 0" class="loading-indicator">
      Cargando datos del gráfico...
    </div>
    <!-- Chart type is now mixed (line and scatter), Chart.js handles this automatically -->
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