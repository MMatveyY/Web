<template>
  <div>
    <canvas ref="chartCanvas" :width="width" :height="height"></canvas>
  </div>
</template>

<script>
export default {
  props: ["data"],
  data() {
    return {
      width: 800,
      height: 400,
      colors: ["#888", "#00f", "#f00", "#0f0", "#000"],
    };
  },
  methods: {
  drawChart() {
    const canvas = this.$refs.chartCanvas;
    const ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, this.width, this.height);

    ['Sum', 'C', 'M', 'Y', 'K'].forEach((channel, index) => {
      const values = this.data[channel];
      const maxValue = Math.max(...values); // Вычисляем максимум для текущего канала
      const path = this.generatePath(values, maxValue); // Передаем максимум

      ctx.fillStyle = this.colors[index];
      ctx.beginPath();
      ctx.moveTo(0, this.height);
      path.forEach(([x, y]) => {
        ctx.lineTo(x, y);
      });
      ctx.lineTo(this.width, this.height);
      ctx.closePath();
      ctx.fill();
    });
  },
  generatePath(values, maxValue) {
    const step = this.width / values.length;
    return values.map((value, i) => {
      const x = i * step;
      const y = this.height - (value / maxValue) * this.height; // Масштабируем значение
      return [x, y];
    });
  },
},

  mounted() {
    this.drawChart();
  },
  watch: {
    data: "drawChart",
  },
};
</script>
