<script>
import { Pie } from 'vue-chartjs';

export default {
  extends: Pie,
  props: {
    data: Object,
    // options: Object,
  },
  mounted() {
    const options = {};
    this.renderChart(this.data, options);
  },
};
</script>
