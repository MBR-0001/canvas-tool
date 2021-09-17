<template>
  <div id="app">
    <div>
      <div>
        Background: <input placeholder="Select background" type="file" @change="updateBackground" accept="image/png, image/jpeg">
      </div>
      <div>
        Aspect ratio: <input v-model="aspectX" min="1" max="30" type="number" style="width: 50px;">:<input v-model="aspectY" min="1" max="30" type="number" style="width: 50px;">
      </div>
    </div>
    <canvas ref="canvas" @mousemove="hover" @mouseleave="hoverLeft" @scroll="hover"
      @mousedown="e => mouseChange(e, true)" @mouseup="e => mouseChange(e, false)"
      style="border: 1px solid black;" :width="width" :height="height"
    >
    </canvas>
    <div>
      X: {{cX}} ({{cX / width}})
      Y: {{cY}} ({{cY / height}})
    </div>
    <div style="margin-top: 10px;">
      <div style="width: 100%; text-align: center;">
        <span>Points:</span> <button @click="clearButton">Clear</button>
      </div>
      <ul
        style="margin: 0; list-style-type: none; padding: 0;"
      >
        <li v-for="(item, i) of dots" v-bind:key="i">
          <span :style="'color: ' + (item.color || 'red') + ';'">• </span>
          <span>X: {{item.x}} ({{item.x / width}}) Y: {{item.y}} ({{item.y / height}})</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      width: 1000,
      aspectX: 16,
      aspectY: 9,
      cX: 0,
      cY: 0,
      mouseDown: false,
      dots: [],
      background: null
    };
  },
  computed: {
    height() {
      return this.width * (this.aspectY / this.aspectX);
    },
    canvas() {
      return this.$refs.canvas ?? {};
    },
    /**
     * @returns {CanvasRenderingContext2D}
     */
    ctx() {
      return this.canvas?.getContext("2d");
    }
  },
  watch: {
    cX() {
      this.refreshCanvas();
    },
    cY() {
      this.refreshCanvas();
    },
    mouseDown() {
      this.refreshCanvas();
    },
    dots() {
      this.refreshCanvas();
    }
  },
  methods: {
    clearButton() {
      this.dots = [];
    },
    updateBackground(event) {
      const img = new Image();
      img.onload = () => {
        this.background = img;
        this.refreshCanvas();
      };
      img.src = URL.createObjectURL(event.target.files[0]);
    },
    scroll() {
      this.cX = this.cY = 0;
    },
    refreshCanvas() {
      const ctx = this.ctx;
      const c = this.canvas;

      ctx.strokeStyle = this.mouseDown ? "black" : "gray";

      ctx.clearRect(0, 0, c.width, c.height);

      if (this.background) {
        ctx.drawImage(this.background, 0, 0, c.height, c.width);
      }

      ctx.beginPath();
      ctx.moveTo(this.cX, 0);
      ctx.lineTo(this.cX, c.width);
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(0, this.cY);
      ctx.lineTo(c.width, this.cY);
      ctx.stroke();

      const prev = ctx.fillStyle;

      for (const dot of this.dots) {
        ctx.fillStyle = dot.color;
        ctx.fillRect(dot.x, dot.y, 5, 5);
      }

      ctx.fillStyle = prev;
    },
    mouseChange(event, up) {
      event.preventDefault();

      this.mouseDown = up;

      if (!up) {
        const color = "#" + Math.floor(Math.random() * 16777215).toString(16);
        this.dots.push({ x: this.cX, y: this.cY, color });
      }
    },
    hoverLeft() {
      this.cX = this.cY = 0;
      this.ctx.clearRect(0, 0, this.canvas?.width, this.canvas?.height);
    },
    hover(event) {
      event.preventDefault();
      const c = this.$refs.canvas;
      if (!c) return;

      this.cX = event.clientX - c.offsetLeft + window.scrollX;
      this.cY = event.clientY - c.offsetTop + window.scrollY;
    }
  },
  created() {
    window.addEventListener("scroll", this.scroll);
  },
  destroyed() {
    window.removeEventListener("scroll", this.scroll);
  }
};
</script>

<style>
#app {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}
</style>
