<script>
import { Line } from 'vue-chartjs';

export default {

  extends: Line,
  props: {
    data: Object,
    // options: Object,
  },
  mounted() {
    const ctx = document.getElementById('line-chart').getContext('2d');
    const gradient = ctx.createLinearGradient(0, 230, 0, 50);
    gradient.addColorStop(1, 'rgba(76, 211, 150, 0.3)');
    gradient.addColorStop(0.5, 'rgba(53, 183, 125, 0.05)');
    gradient.addColorStop(0, 'rgba(119,52,169,0.01)');
    this.data.datasets[0].backgroundColor = gradient;
    this.data.datasets[0].fill = true;
    this.data.datasets[0].borderColor = '#42b883';
    this.data.datasets[0].borderWidth = 2;
    this.data.datasets[0].borderDash = [];
    this.data.datasets[0].borderDashOffset = 0.0;
    this.data.datasets[0].pointBackgroundColor = '#42b883';
    this.data.datasets[0].pointBorderColor = 'rgba(255,255,255,0)';
    this.data.datasets[0].pointHoverBackgroundColor = '#42b883';
    this.data.datasets[0].pointBorderWidth = 20;
    this.data.datasets[0].pointHoverRadius = 4;
    this.data.datasets[0].pointHoverBorderWidth = 15;
    this.data.datasets[0].pointRadius = 4;
    // this.data.datasets[0].label = 'My First dataset';
    const options = {
      maintainAspectRatio: false,
      legend: {
        display: false,
      },
      responsive: true,
      tooltips: {
        backgroundColor: '#f5f5f5',
        titleFontColor: '#333',
        bodyFontColor: '#666',
        bodySpacing: 4,
        xPadding: 12,
        mode: 'nearest',
        intersect: 0,
        position: 'nearest',
      },
      scales: {
        yAxes: [{
          barPercentage: 1.6,
          gridLines: {
            drawBorder: false,
            color: 'rgba(29,140,248,0.0)',
            zeroLineColor: 'transparent',
          },
          ticks: {
            // suggestedMin: Math.min(...this.data.datasets[0].data) - 1,
            suggestedMax: Math.max(...this.data.datasets[0].data) + 1,
            padding: 20,
            fontColor: '#9e9e9e',
          },
        }],

        xAxes: [{
          barPercentage: 1.6,
          gridLines: {
            drawBorder: false,
            color: 'rgba(0,242,195,0.1)',
            zeroLineColor: 'transparent',
          },
          ticks: {
            padding: 20,
            fontColor: '#9e9e9e',
          },
        }],
      },
    };
    this.renderChart(this.data, options);
  },
};
</script>
