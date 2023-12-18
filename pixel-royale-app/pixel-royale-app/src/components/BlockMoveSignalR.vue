<template>
  <canvas ref="canvas" width="800" height="600"></canvas>
</template>

<script>
import * as signalR from "@microsoft/signalr";

export default {
  data() {
    return {
      connection: null,
      ctx: null,
      box: { x: 100, y: 100, size: 50 },
      userId: "user-" + Math.random().toString(36).substring(2, 15),
    };
  },
  mounted() {
    this.initCanvas();
    this.initSignalR();
    window.addEventListener("keydown", this.handleKeyDown);
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.handleKeyDown);
  },
  methods: {
    initCanvas() {
      const canvas = this.$refs.canvas;
      this.ctx = canvas.getContext("2d");
      this.drawBox();
    },
    drawBox() {
      this.ctx.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
      this.ctx.fillStyle = "blue";
      this.ctx.fillRect(this.box.x, this.box.y, this.box.size, this.box.size);
    },
    handleKeyDown(e) {
      switch (e.key) {
        case "ArrowLeft":
          this.box.x -= 10;
          break;
        case "ArrowRight":
          this.box.x += 10;
          break;
        case "ArrowUp":
          this.box.y -= 10;
          break;
        case "ArrowDown":
          this.box.y += 10;
          break;
      }
      this.drawBox();
      this.connection.invoke("MoveBox", this.userId, this.box.x, this.box.y);
    },
    initSignalR() {
      this.connection = new signalR.HubConnectionBuilder()
        .withUrl("/gameHub")
        .build();

      this.connection.on("BoxMoved", (userId, x, y) => {
        if (userId !== this.userId) {
          // Update the position of other users' boxes
          // For simplicity, this example assumes a single box
          // In a real application, you would track multiple boxes
          this.box.x = x;
          this.box.y = y;
          this.drawBox();
        }
      });

      this.connection.start().catch(err => console.error(err.toString()));
    },
  },
};
</script>
