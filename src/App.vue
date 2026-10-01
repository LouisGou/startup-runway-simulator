<script setup>
import { computed, ref } from 'vue'
import CashFlowChart from './components/CashFlowChart.vue'

const cash = ref(120000)
const income = ref(10000)
const expenses = ref(20000)
const hireCost = ref(0)
const hireMonth = ref(3)
const valid = computed(
  () =>
    [cash.value, income.value, expenses.value, hireCost.value].every(
      (value) => typeof value === 'number' && Number.isFinite(value) && value >= 0,
    ) &&
    Number.isInteger(hireMonth.value) &&
    hireMonth.value >= 1 &&
    hireMonth.value <= 12,
)

function buildForecast(monthlyHireCost) {
  if (!valid.value) return []

  let balance = cash.value

  return Array.from({ length: 12 }, (_, index) => {
    const month = index + 1
    const openingCash = balance
    const additionalCost = month >= hireMonth.value ? monthlyHireCost : 0
    const totalExpenses = expenses.value + additionalCost
    const closingCash = openingCash + income.value - totalExpenses

    balance = closingCash

    return {
      month,
      openingCash,
      income: income.value,
      expenses: totalExpenses,
      closingCash,
    }
  })
}

const baselineForecast = computed(() => buildForecast(0))
const forecast = computed(() => buildForecast(hireCost.value))

function describeRunway(rows) {
  if (!valid.value) return 'Enter valid amounts'
  if (cash.value === 0) return 'No cash reserve'
  const depletedMonth = rows.find((row) => row.closingCash <= 0)
  if (!depletedMonth) return 'Cash remains positive through month 12'
  return depletedMonth.closingCash === 0
    ? `Cash reaches zero at the end of month ${depletedMonth.month}`
    : `Cash runs out during month ${depletedMonth.month}`
}

const baselineRunway = computed(() => describeRunway(baselineForecast.value))
const runway = computed(() => describeRunway(forecast.value))

function runwayMonths(rows) {
  if (cash.value === 0) return 0
  const depletedMonth = rows.find((row) => row.closingCash <= 0)
  if (!depletedMonth) return null
  const monthlyBurn = depletedMonth.expenses - depletedMonth.income
  return depletedMonth.month - 1 + depletedMonth.openingCash / monthlyBurn
}

const hiringImpact = computed(() => {
  if (!valid.value) return ''
  if (hireCost.value === 0) return 'No additional hiring cost: both scenarios are identical.'

  const noHireMonths = runwayMonths(baselineForecast.value)
  const withHireMonths = runwayMonths(forecast.value)
  if (noHireMonths !== null && withHireMonths !== null) {
    const reduction = noHireMonths - withHireMonths
    if (reduction > 0) {
      return `Hiring shortens estimated runway by ${formatAmount(reduction)} months.`
    }
  }
  if (noHireMonths === null && withHireMonths !== null) {
    return 'Hiring brings the cash-out date within the 12-month forecast.'
  }
  return 'Compare the month-12 balances to see the cost of hiring over this forecast.'
})

const hiringCashCost = computed(() => {
  if (!valid.value) return 0
  return baselineForecast.value[11].closingCash - forecast.value[11].closingCash
})

function formatAmount(amount) {
  return amount.toLocaleString(undefined, {
    minimumFractionDigits: 0,
    maximumFractionDigits: 2,
  })
}
</script>

<template>
  <main>
    <h1>Startup runway calculator</h1>
    <p>See how long your cash could last.</p>

    <label>
      Starting cash ($)
      <input v-model.number="cash" type="number" min="0" />
    </label>

    <label>
      Monthly cash received ($)
      <input v-model.number="income" type="number" min="0" />
    </label>

    <label>
      Monthly cash expenses ($)
      <input v-model.number="expenses" type="number" min="0" />
    </label>

    <section>
      <h2>Hiring scenario</h2>
      <p>
        Add the employee's total monthly cost, including benefits. Enter 0 for no additional hire.
      </p>
      <label>
        Additional monthly employee cost ($)
        <input v-model.number="hireCost" type="number" min="0" />
      </label>
      <label>
        Start month (1–12)
        <input v-model.number="hireMonth" type="number" min="1" max="12" step="1" />
      </label>
    </section>

    <section v-if="valid" aria-labelledby="comparison-heading">
      <h2 id="comparison-heading">Compare scenarios</h2>
      <div class="scenario-summaries">
        <article>
          <h3>No hire</h3>
          <p>{{ baselineRunway }}</p>
          <p>
            Cash at month 12: <strong>${{ formatAmount(baselineForecast[11].closingCash) }}</strong>
          </p>
        </article>
        <article>
          <h3>With hire</h3>
          <p>{{ runway }}</p>
          <p>
            Cash at month 12: <strong>${{ formatAmount(forecast[11].closingCash) }}</strong>
          </p>
        </article>
      </div>
      <div aria-live="polite">
        <p>{{ hiringImpact }}</p>
        <p v-if="hireCost > 0">
          Hiring adds ${{ formatAmount(hiringCashCost) }} in expenses over these 12 months.
        </p>
      </div>
    </section>
    <p>
      Assumes constant monthly receipts and existing expenses, with the full hiring cost added from
      the start month. Money flows evenly within each month.
    </p>

    <section>
      <h2>12-month cash forecast</h2>
      <p>All amounts use the same dollar currency as your inputs.</p>

      <p v-if="!valid" role="alert">
        Enter non-negative amounts and a whole-number start month from 1 to 12.
      </p>

      <section v-if="valid">
        <h2>Cash balance: no hire vs. with hire</h2>
        <CashFlowChart :forecast="forecast" :baseline-forecast="baselineForecast" />
      </section>

      <div v-if="valid" class="table-container">
        <table>
          <caption>
            Monthly cash flows with the hire, compared with the no-hire closing balance
          </caption>
          <thead>
            <tr>
              <th scope="col">Month</th>
              <th scope="col">Opening cash</th>
              <th scope="col">Cash in</th>
              <th scope="col">Cash out</th>
              <th scope="col">Closing cash: with hire</th>
              <th scope="col">Closing cash: no hire</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="row in forecast" :key="row.month">
              <th scope="row">{{ row.month }}</th>
              <td>{{ formatAmount(row.openingCash) }}</td>
              <td>{{ formatAmount(row.income) }}</td>
              <td>{{ formatAmount(row.expenses) }}</td>
              <td :class="{ negative: row.closingCash < 0 }">
                {{ formatAmount(row.closingCash) }}
              </td>
              <td :class="{ negative: baselineForecast[row.month - 1].closingCash < 0 }">
                {{ formatAmount(baselineForecast[row.month - 1].closingCash) }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <p>Negative balances show the funding gap if spending continues.</p>
    </section>
  </main>
</template>

<style scoped>
main {
  max-width: 960px;
  margin: 48px auto;
  padding: 24px;
  font-family: system-ui, sans-serif;
}

label {
  display: block;
  margin: 20px 0;
}

input {
  display: block;
  box-sizing: border-box;
  width: 100%;
  margin-top: 8px;
  padding: 10px;
  font: inherit;
}

section {
  margin-top: 40px;
}

.scenario-summaries {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 240px), 1fr));
  gap: 16px;
}

.scenario-summaries article {
  padding: 16px;
  border: 1px solid #888;
  border-radius: 8px;
}

.scenario-summaries h3 {
  margin-top: 0;
}

caption {
  text-align: left;
  padding: 12px 0;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 12px;
  text-align: right;
  border-bottom: 1px solid #888;
  white-space: nowrap;
}

th:first-child {
  text-align: left;
}

.negative {
  color: #d33;
  font-weight: 700;
}
</style>
