<script setup>
import { computed } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Tooltip,
  Legend,
} from 'chart.js'

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Tooltip, Legend)

const props = defineProps({
  forecast: {
    type: Array,
    required: true,
  },
  baselineForecast: {
    type: Array,
    required: true,
  },
})

const chartData = computed(() => ({
  labels: ['Start', ...props.forecast.map((row) => `Month ${row.month}`)],
  datasets: [
    {
      label: 'No hire',
      data: [
        props.baselineForecast[0]?.openingCash ?? 0,
        ...props.baselineForecast.map((row) => row.closingCash),
      ],
      borderColor: '#64748b',
      backgroundColor: '#64748b',
      borderDash: [6, 4],
      borderWidth: 3,
      pointRadius: 3,
      tension: 0,
    },
    {
      label: 'With hire',
      data: [props.forecast[0]?.openingCash ?? 0, ...props.forecast.map((row) => row.closingCash)],
      borderColor: '#2563eb',
      backgroundColor: '#2563eb',
      borderWidth: 3,
      pointRadius: 3,
      tension: 0,
    },
  ],
}))

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  interaction: {
    mode: 'index',
    intersect: false,
  },
  plugins: {
    legend: {
      display: true,
      position: 'bottom',
    },
    tooltip: {
      callbacks: {
        label: (context) =>
          `${context.dataset.label}: $${context.parsed.y.toLocaleString(undefined, { maximumFractionDigits: 2 })}`,
      },
    },
  },
  scales: {
    x: {
      title: {
        display: true,
        text: 'Forecast month',
      },
    },
    y: {
      beginAtZero: true,
      title: {
        display: true,
        text: 'Cash balance ($)',
      },
    },
  },
}
</script>

<template>
  <div class="chart-container">
    <Line
      :data="chartData"
      :options="chartOptions"
      aria-label="No hire (dashed line) and with hire (solid line) cash balances over 12 months. Exact figures appear in the comparison table below."
    />
  </div>
</template>

<style scoped>
.chart-container {
  position: relative;
  height: 360px;
  width: 100%;
}
</style>
