<script setup>
import { ref, onMounted } from 'vue';
import Papa from 'papaparse';
import PriceChart from './charts/PriceChart.vue';
import VolumeChart from './charts/VolumeChart.vue';
import SignalDistribution from './charts/SignalDistribution.vue';

const originalData = ref([]);
const featuresData = ref([]);

const loadData = async () => {
  try {
    // Cargar datos originales
    const originalResponse = await fetch('../../data/pepeusdt_5m.csv');
    const originalCsv = await originalResponse.text();
    const originalParsed = Papa.parse(originalCsv, { header: true });
    originalData.value = originalParsed.data.map(row => ({
      ...row,
      timestamp: new Date(row.timestamp),
      close: parseFloat(row.close),
      volume: parseFloat(row.volume)
    })).slice(0, 100); // últimas 100 velas

    // Cargar datos de características
    const featuresResponse = await fetch('../../data/pepeusdt_5m_features.csv');
    const featuresCsv = await featuresResponse.text();
    const featuresParsed = Papa.parse(featuresCsv, { header: true });
    featuresData.value = featuresParsed.data;
  } catch (error) {
    console.error('Error cargando datos:', error);
  }
};

onMounted(() => {
  loadData();
});
</script>

<template>
  <div class="trading-dashboard">
    <h1>Dashboard de Trading PEPE/USDT</h1>
    <div class="charts-container">
      <div class="chart">
        <PriceChart :data="originalData" />
      </div>
      <div class="chart">
        <VolumeChart :data="originalData" />
      </div>
      <div class="chart">
        <SignalDistribution :data="featuresData" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.trading-dashboard {
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

h1 {
  text-align: center;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.charts-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.chart {
  background: white;
  border-radius: 8px;
  padding: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
