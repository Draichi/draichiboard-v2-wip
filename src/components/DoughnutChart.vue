<script>
import { Doughnut } from 'vue-chartjs';

export default {
  extends: Doughnut,
  props: {
    data: Object,
    // options: Object,
  },
  mounted() {
    const options = {
      responsive: true,
      legend: {
        position: 'bottom',
        // align: 'start',
      },
    };
    // const ctx = document.getElementById('doughnut-chart').getContext('2d');
    // const gradient = ctx.createRadialGradient(300, 300, 1, 300, 300, 300);
    // gradient.addColorStop(0, 'rgba(76, 211, 150, 0.3)');
    // gradient.addColorStop(1, 'rgba(119,52,169,0.01)');
    // this.data.datasets[0].backgroundColor = gradient;
    this.renderChart(this.data, options);
  },
};
</script>
