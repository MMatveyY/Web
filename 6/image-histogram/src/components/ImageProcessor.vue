<template>
  <div class="image-analyzer">
    <div class="card">
      <p>Выберите изображение из файла</p>
      <input type="file" @change="handleFileChange" accept="image/jpeg" />
      <br />
      <canvas ref="imageCanvas" width="500" height="500"></canvas>
    </div>
    <div class="card">
      <p>Гистограмма</p>
      <div>
        <input type="radio" value="value" v-model="histogramMode" @change="drawHistogram" />
        <label>Значение</label>
        <input type="radio" value="color" v-model="histogramMode" @change="drawHistogram" />
        <label>Цвет</label>
      </div>
      <canvas ref="histogramCanvas" width="256" height="150" style="border: solid 1px #999;"></canvas>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      histogramMode: 'value',
      imageData: null,
      histogramData: {
        cyan: new Array(256).fill(0),
        magenta: new Array(256).fill(0),
        yellow: new Array(256).fill(0),
        black: new Array(256).fill(0),
      },
    };
  },
  methods: {
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        const img = new Image();
        img.src = URL.createObjectURL(file);
        img.onload = () => {
          this.processImage(img);
        };
      }
    },
    processImage(img) {
      const canvas = this.$refs.imageCanvas;
      const ctx = canvas.getContext("2d");
      canvas.width = 500;
      canvas.height = 500;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      this.imageData = imageData.data;

      this.applyInversion(imageData);
      this.calculateHistogram();
      this.drawHistogram();
    },
    applyInversion(imageData) {
      for (let i = 0; i < imageData.data.length; i += 4) {
        imageData.data[i] = 255 - imageData.data[i]; // R
        imageData.data[i + 1] = 255 - imageData.data[i + 1]; // G
        imageData.data[i + 2] = 255 - imageData.data[i + 2]; // B
      }
      const canvas = this.$refs.imageCanvas;
      const ctx = canvas.getContext("2d");
      ctx.putImageData(imageData, 0, 0);
    },
    calculateHistogram() {
      this.histogramData.cyan.fill(0);
      this.histogramData.magenta.fill(0);
      this.histogramData.yellow.fill(0);
      this.histogramData.black.fill(0);

      for (let i = 0; i < this.imageData.length; i += 4) {
        const r = this.imageData[i];
        const g = this.imageData[i + 1];
        const b = this.imageData[i + 2];

        const c = 255 - r;
        const m = 255 - g;
        const y = 255 - b;
        const k = Math.min(c, m, y);

        this.histogramData.cyan[c]++;
        this.histogramData.magenta[m]++;
        this.histogramData.yellow[y]++;
        this.histogramData.black[k]++;
      }
    },
    drawHistogram() {
      const canvas = this.$refs.histogramCanvas;
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Если режим "Цвет"
      if (this.histogramMode === 'color') {
        const combined = this.histogramData.cyan.map((v, i) =>
          v + this.histogramData.magenta[i] + this.histogramData.yellow[i] + this.histogramData.black[i]
        );
        const maxValue = Math.max(...combined); // Максимум для комбинированного канала
        this.drawChannel(combined, "gray", maxValue, ctx, canvas);
      } else {
        // Если режим "Значение"
        ['cyan', 'magenta', 'yellow', 'black'].forEach((channel, index) => {
          const maxValue = Math.max(...this.histogramData[channel]); // Максимум для каждого канала
          const color = ['cyan', 'magenta', 'yellow', 'black'][index];
          this.drawChannel(this.histogramData[channel], color, maxValue, ctx, canvas);
        });
      }
    },
    drawChannel(data, color, maxValue, ctx, canvas) {
      ctx.fillStyle = color;
      data.forEach((value, i) => {
        const height = (value / maxValue) * canvas.height; // Масштабируем значение
        ctx.fillRect(i, canvas.height - height, 1, height);
      });
    },
  },
};
</script>

<style>
.image-analyzer {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.card {
  margin: 20px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
</style>
