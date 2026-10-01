# Startup Runway Simulator

A Vue app for exploring how a planned hire changes a startup's cash balance and runway. Adjust the financial assumptions to compare a no-hire baseline with a hiring scenario across 12 months.

## Features

- Enter starting cash, monthly cash receipts, and existing monthly expenses.
- Set an employee's total monthly cost and start month.
- Compare both scenarios on an interactive chart and in a monthly cash-flow table.
- See each scenario's projected cash-out month, month-12 balance, and the additional hiring cost over the forecast.
- Validate non-negative financial inputs and a hiring start month from 1 to 12.

Built with Vue, JavaScript, Vite, Chart.js, and vue-chartjs. Calculations run in the browser; no backend or account is required. Inputs currently reset on refresh.

## Run locally

Use Node.js matching the version range in `package.json` (`^22.18.0 || >=24.12.0`) and npm.

From the project directory:

```sh
npm ci
npm run dev
```

Open the local URL printed in the terminal. To build and preview the production version:

```sh
npm run build
npm run preview
```

The production files are generated in `dist/`. `npm run lint` runs the configured linters and applies automatic fixes.

## Example: hiring in month 3

Set starting cash to **$120,000**, monthly cash receipts to **$10,000**, and existing monthly expenses to **$20,000**. Add a hire costing **$5,000 per month**, starting in **month 3**.

| Result                      | No hire         | With hire      |
| --------------------------- | --------------- | -------------- |
| Cash-out point              | End of month 12 | During month 9 |
| Cash at the end of month 12 | $0              | -$50,000       |

Under the timing assumptions below, the hire reduces estimated runway from 12 to about 8.67 months, a reduction of about **3.33 months**. The extra expense is **$50,000** across months 3–12.

## Financial model and assumptions

Each month uses:

```text
Closing cash = Opening cash + Cash received - Cash paid
Next month's opening cash = This month's closing cash
```

- Cash receipts and existing expenses remain constant throughout the forecast.
- The full employee cost is added every month from the selected start month, including that month. Enter the total employer cost, including benefits; avoid counting it again in existing expenses.
- Hiring changes costs only; it does not automatically increase sales or cash receipts.
- Cash comes in and goes out evenly within each month. Fractional runway estimates interpolate the point where cash reaches zero using that month's net cash burn.
- Use one currency consistently. The dollar symbol is a display convention; the app does not convert currencies.
- Negative balances show the funding gap if spending continues; the model does not automatically borrow or raise funding.
- The horizon is **12 months**. A positive balance at month 12 does not mean indefinite runway, and cash-out dates beyond that horizon are not calculated.
- This version does not model revenue growth, delayed collections, financing, taxes, or one-time expenses separately.

## Project structure

```text
src/App.vue                       Inputs, forecast calculations, summaries, and table
src/components/CashFlowChart.vue   Interactive scenario comparison chart
src/main.js                       Vue application entry point
```
