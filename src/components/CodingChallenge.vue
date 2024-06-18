<template>
  <div>
    <canvas id="riskRewardChart"></canvas>
    <div>
      <p>Max Profit: {{ maxProfit }}</p>
      <p>Max Loss: {{ maxLoss }}</p>
      <p>Break Even Points: {{ breakEvenPoints }}</p>
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
export default {
  name: 'CodingChallenge',
  props: {
    optionsData: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      maxProfit: null,
      maxLoss: null,
      breakEvenPoints: [],
      chart: null
    }
  },
  methods: {
    calculateRiskReward() {
      // Initialize variables for maximum profit, maximum loss, and break-even points
      let maxProfit = -Infinity;
      let maxLoss = Infinity;
      let breakEvenPoints = [];

      // Iterate over each option in the optionsData array
      this.optionsData.forEach(option => {
        let profit, loss, breakEvenPoint;

        // Calculate profit, loss, and break-even point based on the type of option and whether it's long or short
        if (option.type === 'Call') {
          if (option.long_short === 'long') {
            profit = (option.strike_price - option.bid > 0) ? option.strike_price - option.bid : 0;
            loss = -option.bid;
            breakEvenPoint = option.strike_price + option.bid;
          } else {
            profit = option.ask;
            loss = (option.strike_price - option.ask > 0) ? -(option.strike_price - option.ask) : 0;
            breakEvenPoint = option.strike_price + option.ask;
          }
        } else if (option.type === 'Put') {
          if (option.long_short === 'long') {
            profit = option.strike_price - option.bid;
            loss = -option.bid;
            breakEvenPoint = option.strike_price - option.bid;
          } else {
            profit = option.ask;
            loss = (option.strike_price - option.ask > 0) ? option.strike_price - option.ask : 0;
            breakEvenPoint = option.strike_price - option.ask;
          }
        }

        // Assign the calculated profit and loss to the option object
        option.profit = profit;
        option.loss = loss;

        // Update the maximum profit and loss
        maxProfit = Math.max(maxProfit, profit);
        maxLoss = Math.min(maxLoss, loss);

        // Add the break-even point to the array
        breakEvenPoints.push(breakEvenPoint);
      });

      // Assign the calculated values to the component's data properties
      this.maxProfit = maxProfit;
      this.maxLoss = maxLoss;
      this.breakEvenPoints = breakEvenPoints;
    },
    drawChart() {
      if (this.chart) {
        this.chart.destroy();
      }
      const ctx = document.getElementById('riskRewardChart').getContext('2d');
      Chart.register(...registerables);
      this.chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: this.optionsData.map(option => option.strike_price),
          datasets: [{
            label: 'Profit',
            data: this.optionsData.map(option => option.profit),
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            borderWidth: 1
          }, {
            label: 'Loss',
            data: this.optionsData.map(option => option.loss),
            backgroundColor: 'rgba(255, 99, 132, 0.2)',
            borderColor: 'rgba(255, 99, 132, 1)',
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            y: {
              type: 'linear',
              beginAtZero: true
            }
          }
        }
      });
    }
  },
  mounted() {
    this.calculateRiskReward();
    this.drawChart();
  },
  watch: {
    optionsData: {
      handler() {
        this.calculateRiskReward();
        this.drawChart();
      },
      deep: true
    }
  }
}
</script>

<style scoped>
/* No styles yet */
</style>
