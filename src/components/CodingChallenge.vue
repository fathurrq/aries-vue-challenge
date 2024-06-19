<template>
  <div class="wrapper">
    <h1>Options Profit Calculator</h1>
    <div class="main-content">
      <div class="form-wrapper">
        <h2>Enter Options Detail</h2>
        <form @submit.prevent="calculateRiskReward">
          <div v-for="(option, index) in optionsData" :key="index">
            <label>
              Option {{ index + 1 }} Type:
              <select v-model="option.type">
                <option disabled value="">Select type</option>
                <option>Call</option>
                <option>Put</option>
              </select>
            </label>
            <label>
              Strike Price:
              <input
                type="text"
                @keypress="allowNumeric($event)"
                v-model.number="option.strike_price"
                placeholder="Strike Price"
              />
            </label>
            <label>
              Option Price:
              <input
                type="text"
                @keypress="allowNumeric($event)"
                v-model.number="option.ask"
                placeholder="Option Price"
              />
            </label>
          </div>
          <button type="submit" class="calculate-btn">Calculate</button>
        </form>
      </div>
      <div class="dashboard">
        <canvas ref="mainChart"></canvas>
        <div class="cards-group" v-if="result">
          <div class="card">
            <h3>Max Profit</h3>
            <p>{{ result.maxProfit }}</p>
          </div>
          <div class="card">
            <h3>Max Loss</h3>
            <p>{{ result.maxLoss }}</p>
          </div>
          <div class="card">
            <h3>Break Even Points</h3>
            <p v-if="result.breakEvenPoints.length === 0">-</p>
            <p v-else>{{ result.breakEvenPoints.join(", ") }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from "chart.js/auto";

export default {
  name: "CodingChallenge",
  props: {
    optionsData: {
      require: true,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      result: null,
      chart: null,
    };
  },
  methods: {
    calculateRiskReward() {
      const { data, maxProfit, maxLoss, breakEvenPoints } =
        this.getRiskReward();

      this.result = { maxProfit, maxLoss, breakEvenPoints };

      if (this.chart) {
        this.chart.destroy();
      }

      const ctx = this.$refs.mainChart.getContext("2d");
      this.chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: data.x,
          datasets: [
            {
              label: "Profit/Loss",
              data: data.y,
              borderColor: "rgba(10, 105, 233, 1)",
              borderWidth: 2,
              fill: false,
            },
          ],
        },
        options: {
          responsive: true,
          scales: {
            x: {
              title: {
                display: true,
                text: "Price",
              },
            },
            y: {
              title: {
                display: true,
                text: "Profit/Loss",
              },
            },
          },
        },
      });
    },
    getRiskReward() {
      const { minPrice, maxPrice } = this.calculatePriceRange();
      const steps = 50;
      const priceStep = (maxPrice - minPrice) / steps;
      const underlyingPrices = Array.from({ length: steps + 1 }, (_, i) =>
        (minPrice + i * priceStep).toFixed(2)
      );
      const profitLosses = underlyingPrices.map((price) =>
        this.calculateTotalProfitLoss(price)
      );

      const maxProfit = Math.max(...profitLosses).toFixed(2);
      const maxLoss = Math.min(...profitLosses).toFixed(2);
      const breakEvenPoints = underlyingPrices.filter(
        (price, idx) => profitLosses[idx] === 0
      );

      return {
        data: { x: underlyingPrices, y: profitLosses },
        maxProfit,
        maxLoss,
        breakEvenPoints,
      };
    },

    calculatePriceRange() {
      const strikes = this.optionsData.map((opt) =>
        parseFloat(opt.strike_price)
      );
      const minPrice = Math.min(...strikes) * 0.5;
      const maxPrice = Math.max(...strikes) * 1.5;
      return { minPrice, maxPrice };
    },

    calculateTotalProfitLoss(underlyingPrice) {
      return this.optionsData.reduce((totalProfitLoss, option) => {
        const cost = (parseFloat(option.bid) + parseFloat(option.ask)) / 2;
        const profitLoss = this.calculateProfitLoss(
          option,
          underlyingPrice,
          cost
        );
        return totalProfitLoss + profitLoss;
      }, 0);
    },
    calculateProfitLoss(option, underlyingPrice, cost) {
      const priceDifference = underlyingPrice - option.strike_price;
      if (option.type.toLowerCase() === "call") {
        return option.long_short === "long"
          ? Math.max(priceDifference, 0) - cost
          : cost - Math.max(priceDifference, 0);
      } else {
        return option.long_short === "short"
          ? Math.max(-priceDifference, 0) - cost
          : cost - Math.max(-priceDifference, 0);
      }
    },
    allowNumeric(event) {
      if (
        !/[0-9.]/.test(event.key) ||
        (event.key === "." && event.target.value.includes("."))
      ) {
        event.preventDefault();
      }
    },
  },
};
</script>

<style scoped>
.wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1,
h2 {
  color: #333;
}

label {
  display: flex;
  align-items: center;
  margin: 10px 0;
}

select,
input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  margin-left: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button.calculate-btn {
  background-color: #4caf50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 10px;
}

button.calculate-btn:hover {
  background-color: #45a049;
}

.main-content {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.form-wrapper {
  flex: 1 1 300px;
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.dashboard {
  flex: 2 1 600px;
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  justify-content: space-around;
}

.cards-group {
  display: flex;
  gap: 10px;
  margin-top: 30px;
}
@media(max-width: 760px) {
  .cards-group {
    flex-direction: column;
}
}

.card {
  flex-grow: 1;
  padding: 15px;
  background: #f0f0f0;
  border-radius: 5px;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.05);
}

canvas {
  width: 100%;
  height: 400px;
}
</style>
