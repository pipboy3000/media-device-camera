<template>
  <div id="app">
    <div class="camera" v-show="showVideo">
      <video ref="video" playsinline autoplay muted />
      <button @click="onScreenshot">写真を撮る</button>
    </div>
    <div class="photo" v-show="!showVideo">
      <canvas ref="canvas" />
      <button @click="onBack">撮影に戻る</button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import "ress";

const initialConstraints = {
  audio: false,
  video: {
    facingMode: "environment"
  }
};

export default Vue.extend({
  name: "app",
  data() {
    return {
      showVideo: true
    };
  },
  async mounted() {
    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      const videoDevices = devices.filter(
        device => device.kind === "videoinput"
      );

      const constraints = {
        ...initialConstraints,
        deviceId: {
          exact: videoDevices[0].deviceId
        }
      };

      const video = this.$refs.video as HTMLVideoElement;
      video.srcObject = await navigator.mediaDevices.getUserMedia(constraints);
    } catch (error) {
      console.error(error);
    }
  },
  methods: {
    onScreenshot: async function() {
      const canvas = this.$refs.canvas as HTMLCanvasElement;
      const video = this.$refs.video as HTMLVideoElement;
      if (canvas) {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const context2d = canvas.getContext("2d");
        if (context2d) {
          context2d.drawImage(video, 0, 0);
          this.showVideo = false;
        }
      }
    },
    onBack() {
      this.showVideo = true;
    }
  }
});
</script>

<style lang="scss">
body {
  background: #333;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  .camera,
  .photo {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  button {
    padding: 14px 24px;
    border-radius: 4px;
    background: transparent;
    color: #fff;
    border: 3px solid #fff;
  }

  video,
  canvas {
    width: 100%;
    height: auto;
    margin-bottom: 30px;
  }
}
</style>
