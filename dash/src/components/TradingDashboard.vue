<script setup>
import { ref, onMounted } from 'vue';
import Papa from 'papaparse';
import PriceChart from './charts/PriceChart.vue';
import VolumeChart from './charts/VolumeChart.vue';
// SignalDistribution import removed

const originalData = ref([]);
const featuresData = ref([]);

// Store the raw fetched data to avoid re-fetching
const rawOriginalData = ref([]); // This will now hold all original candle data

const processOriginalDataRow = (row) => {
  const timestamp = new Date(row.timestamp);
  if (isNaN(timestamp.getTime())) {
    console.warn(`Invalid timestamp in original data: ${row.timestamp}. Skipping row.`);
    return null;
  }

  const close = parseFloat(row.close);
  if (isNaN(close)) {
    console.warn(`Invalid close value in original data: ${row.close}. Using 0 as fallback.`);
  }

  const volume = parseFloat(row.volume);
  if (isNaN(volume)) {
    console.warn(`Invalid volume value in original data: ${row.volume}. Using 0 as fallback.`);
  }

  return {
    ...row,
    timestamp,
    close: isNaN(close) ? 0 : close,
    volume: isNaN(volume) ? 0 : volume,
  };
};

const loadOriginalCandleData = async () => {
  try {
    const response = await fetch('../../data/pepeusdt_5m.csv');
    if (!response.ok) {
      throw new Error(`HTTP error ${response.status} while fetching original data.`);
    }
    const csvText = await response.text();
    let parsedResult;
    try {
      parsedResult = Papa.parse(csvText, { header: true });
    } catch (parseError) {
      console.error('Error parsing original data CSV:', parseError);
      originalData.value = [];
      return;
    }

    if (parsedResult.errors && parsedResult.errors.length > 0) {
      console.warn('Errors encountered during original data CSV parsing:', parsedResult.errors);
    }

    if (!Array.isArray(parsedResult.data)) {
      console.error('Parsed original data is not an array:', parsedResult);
      rawOriginalData.value = [];
    } else {
      // Store all valid processed rows
      rawOriginalData.value = parsedResult.data
        .map(processOriginalDataRow)
        .filter(row => row !== null);
      originalData.value = rawOriginalData.value; // Display all loaded original data
    }
  } catch (error) {
    console.error('Failed to load original candle data:', error.message);
    rawOriginalData.value = [];
    originalData.value = []; // Ensure displayed data is reset on error
  }
};

const loadFeaturesSignalData = async () => {
  try {
    const response = await fetch('../../data/pepeusdt_5m_features.csv');
    if (!response.ok) {
      throw new Error(`HTTP error ${response.status} while fetching features data.`);
    }
    const csvText = await response.text();
    let parsedResult;
    try {
      parsedResult = Papa.parse(csvText, { header: true });
    } catch (parseError) {
      console.error('Error parsing features data CSV:', parseError);
      featuresData.value = [];
      return;
    }

    if (parsedResult.errors && parsedResult.errors.length > 0) {
      console.warn('Errors encountered during features data CSV parsing:', parsedResult.errors);
    }

    if (!Array.isArray(parsedResult.data)) {
      console.error('Parsed features data is not an array:', parsedResult);
      featuresData.value = [];
    } else {
      featuresData.value = parsedResult.data;
    }
  } catch (error) {
    console.error('Failed to load features signal data:', error.message);
    featuresData.value = []; // Ensure data is reset on error
  }
};

import { computed } from 'vue';

const loadAllData = async () => {
  await loadOriginalCandleData(); // Always load full original data
  await loadFeaturesSignalData(); // Always load full features data
};

const signalPoints = computed(() => {
  if (!rawOriginalData.value.length || !featuresData.value.length) {
    return [];
  }

  // rawOriginalData.value already has Date objects for timestamps.
  // We'll create a map from their epoch time for faster lookups.
  const priceDataMap = new Map(
    rawOriginalData.value.map(data => [data.timestamp.getTime(), data.close])
  );

  const points = [];
  for (const signal of featuresData.value) {
    if (signal.label && (signal.label.toUpperCase() === 'BUY' || signal.label.toUpperCase() === 'SELL')) {
      // Assuming signal.timestamp is a string that needs to be converted to a Date object
      const signalDate = new Date(signal.timestamp);
      const signalTime = signalDate.getTime();
      if (isNaN(signalTime)) {
        console.warn(`Invalid timestamp in features data: ${signal.timestamp}. Skipping signal.`);
        continue;
      }

      if (priceDataMap.has(signalTime)) {
        points.push({
          x: signalDate, // Use the Date object for Chart.js
          y: priceDataMap.get(signalTime),
          type: signal.label.toUpperCase(), // Ensure type is consistent (BUY/SELL)
        });
      } else {
         // Optional: console.warn for signals without matching price data
         // console.warn(`No matching price data found for signal at timestamp: ${signal.timestamp} (Epoch: ${signalTime})`);
      }
    }
  }
  return points;
});

onMounted(() => {
  loadAllData(); // Initial data load
});
</script>

<template>
  <div class="trading-dashboard">
    <h1>Dashboard de Trading PEPE/USDT</h1>
    <!-- Controls container removed -->
    <div class="charts-container">
      <div class="chart">
        <!-- Pass signalPoints to PriceChart in a later step -->
        <PriceChart :data="originalData" :signals="signalPoints" />
      </div>
      <div class="chart">
        <VolumeChart :data="originalData" />
      </div>
      <!-- SignalDistribution component usage removed -->
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
  color: var(--vt-c-indigo); /* Using a color from base.css */
}

/* .controls-container styling removed as the container is deleted */

.charts-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.chart {
  background: var(--vt-c-white, white); /* Using CSS var from base.css */
  border-radius: 8px;
  padding: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
