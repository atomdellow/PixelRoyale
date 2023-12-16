<template>
  <div>
    <canvas> </canvas>
  </div>
</template>
<script>
import signalR from "@microsoft/signalr";

export default {
  data() {
    return {
      connection: null,
    };
  },
  mounted() {
    this.startSignalRConnection();
  },
  methods: {
    window() {
      const canvas = document.querySelector("canvas");
      // const c = canvas.getContext("2d");

      canvas.width = 1024;
      canvas.height = 576;
    },

    async startSignalRConnection() {
      this.connection = new signalR.HubConnectionBuilder()
        .withUrl("/gameHub")
        .build();

      this.connection.on("Send", (message) => {
        console.log(message);
      });

      try {
        await this.connection.start();
        console.log("SignalR Connected.");
      } catch (err) {
        console.error(err);
      }
    },

    async joinLobby(lobbyName) {
      if (this.connection) {
        await this.connection.invoke("JoinLobby", lobbyName);
      }
    },
  },
};
</script>
