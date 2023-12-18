<template>
  <canvas ref="canvas" width="800" height="600"></canvas>
</template>

<script>
import { HubConnectionBuilder } from "@microsoft/signalr";

export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      connection: null,
      boxes: {},
      userId: "user-" + Math.random().toString(36).substring(2, 15),
    };
  },
  mounted() {
 
    // this.joinLobby("defaultLobby"); // Replace "defaultLobby" with your lobby name
    this.initCanvas();
    this.boxes[this.userId] = { x: 100, y: 100, size: 50 };
   this.initSignalR();
    window.addEventListener("keydown", this.handleKeyDown);
  },
  beforeUnmount() {
    window.removeEventListener("keydown", this.handleKeyDown);
  },
  methods: {
    initCanvas() {
      this.canvas = this.$refs.canvas;
      if (this.canvas) {
        this.ctx = this.canvas.getContext("2d");
        this.drawboxes();
      }
    },

    // joinLobby(lobbyName) {
    //   if (this.connection && this.connection.state === "Connected") {
    //     this.connection
    //       .invoke("JoinLobby", lobbyName)
    //       .catch((err) => console.error("Error joining lobby: ", err));
    //   } else {
    //     console.log("Connection not established. Cannot join lobby.");
    //   }
    // },

    drawboxes() {
      if (!this.ctx) return;

      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      for (const [userId, boxes] of Object.entries(this.boxes)) {
        this.ctx.fillStyle = userId === this.userId ? "blue" : "red"; // Different color for the current player
        this.ctx.fillRect(boxes.x, boxes.y, boxes.size, boxes.size);
      }
    },
    handleKeyDown(e) {
      if (!this.connection || this.connection.state !== "Connected") {
        console.log("SignalR connection not established.");
        return;
      }
      console.log("Updated: " + toString(this.boxes.x));
      switch (e.key) {
        case "ArrowLeft":
          this.boxes.x -= 10;
          console.log("Updated: " + toString(this.boxes.x));
          break;
        case "ArrowRight":
          this.boxes.x += 10;
          console.log("Updated: " + toString(this.boxes.x));
          break;
        case "ArrowUp":
          this.boxes.y -= 10;
          console.log("Updated: " + toString(this.boxes.y));
          break;
        case "ArrowDown":
          this.boxes.y += 10;
          console.log("Updated: " + toString(this.boxes.y));
          break;
      }
      this.drawboxes();
      this.connection
        .invoke("BoxMoved", this.userId, this.boxes.x, this.boxes.y)
        .catch((err) => console.error("Error sending message: ", err));
    },

    initSignalR() {
      this.connection = new HubConnectionBuilder()
        .withUrl("https://localhost:7099/GameHub")
        .build();
      this.connection.on("BoxMoved", (userId, x, y) => {
        if (!this.boxes[userId]) {
          this.boxes[userId] = { x: 0, y: 0, size: 50 }; // Initialize a new boxes if it doesn't exist
        }
        this.boxes[userId].x = x;
        this.boxes[userId].y = y;
        this.drawboxes();
      });
      this.connection
        .start()
        .then(() => {
          console.log("SignalR Connected");
        })
        .catch((err) => console.error("SignalR Connection Error: ", err));
    },
  },
};
</script>
