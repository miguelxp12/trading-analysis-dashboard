<script setup>
import { ref, onMounted } from 'vue';
import Papa from 'papaparse';
import PriceChart from './charts/PriceChart.vue';
import VolumeChart from './charts/VolumeChart.vue';
import SignalDistribution from './charts/SignalDistribution.vue';

const originalData = ref([]);
const featuresData = ref([]);
const numberOfCandles = ref(100); // Default to 100 candles

// Store the raw fetched data to avoid re-fetching
const rawOriginalData = ref([]);

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
      // Store all valid processed rows first
      rawOriginalData.value = parsedResult.data
        .map(processOriginalDataRow)
        .filter(row => row !== null);
      // Then apply slicing based on numberOfCandles
      applyCandleLimit();
    }
  } catch (error) {
    console.error('Failed to load original candle data:', error.message);
    rawOriginalData.value = [];
    originalData.value = []; // Ensure displayed data is reset on error
  }
};

const applyCandleLimit = () => {
  if (rawOriginalData.value.length === 0) {
    originalData.value = [];
    return;
  }
  const numCandles = parseInt(numberOfCandles.value, 10);
  if (isNaN(numCandles) || numCandles <= 0) {
    // If input is invalid, default to a sensible value or all data
    originalData.value = rawOriginalData.value.slice(0, 100); // Default to 100 or all
    console.warn(`Invalid number of candles: ${numberOfCandles.value}. Defaulting to 100.`);
    numberOfCandles.value = 100; // Reset the input if invalid
  } else if (numCandles > rawOriginalData.value.length) {
    originalData.value = rawOriginalData.value; // Show all available if requested is too high
  } else {
    originalData.value = rawOriginalData.value.slice(0, numCandles);
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

import { watch } from 'vue';

const loadAllData = async () => {
  // Only fetch if raw data is not already loaded
  if (rawOriginalData.value.length === 0) {
    await loadOriginalCandleData();
  } else {
    // If raw data exists, just re-apply slicing
    applyCandleLimit();
  }
  // Features data is independent of numberOfCandles for now
  if (featuresData.value.length === 0) {
    await loadFeaturesSignalData();
  }
};

watch(numberOfCandles, (newValue, oldValue) => {
  const num = parseInt(newValue, 10);
  // Basic validation before re-processing
  if (isNaN(num) || num < 10) {
    numberOfCandles.value = 10; // Enforce minimum
    // console.warn("Number of candles cannot be less than 10.");
    // No need to call applyCandleLimit here, the corrected value will trigger the watcher again.
    return;
  }
  if (num > 5000 && rawOriginalData.value.length > 5000) { // Arbitrary reasonable max if data is huge
     numberOfCandles.value = 5000;
    // console.warn("Number of candles capped at 5000 for performance.");
    return;
  }
  applyCandleLimit();
}, { immediate: false }); // immediate: false, to avoid running on initial setup if not needed before rawOriginalData is populated.

onMounted(() => {
  loadAllData(); // Initial data load
});
</script>

<template>
  <div class="trading-dashboard">
    <h1>Dashboard de Trading PEPE/USDT</h1>
    <div class="controls-container">
      <label for="numberOfCandlesInput">Número de Velas:</label>
      <input
        id="numberOfCandlesInput"
        type="number"
        v-model.number="numberOfCandles"
        min="10"
        max="5000" 
        step="10"
      />
    </div>
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
  color: #2c3e50; /* Using a color from base.css potentially */
}

.controls-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 2rem;
  gap: 0.5rem;
  background-color: var(--color-background-soft, #f8f8f8); /* Using CSS var from base.css */
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.05);
}

.controls-container label {
  font-weight: bold;
  color: var(--color-text, #2c3e50);
}

.controls-container input[type="number"] {
  padding: 0.5rem;
  border: 1px solid var(--color-border, #ccc);
  border-radius: 4px;
  width: 80px; /* Adjust as needed */
  text-align: right;
}

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
