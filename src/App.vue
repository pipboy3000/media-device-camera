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
import * as mobilenet from "@tensorflow-models/mobilenet";

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
      showVideo: true,
      videoTrack: {},
      imageCapture: {}
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
      const mediaStream = await navigator.mediaDevices.getUserMedia(
        constraints
      );
      video.srcObject = mediaStream;
      this.videoTrack = mediaStream.getVideoTracks()[0];
      this.imageCapture = new ImageCapture(this.videoTrack);
    } catch (error) {
      console.error(error);
    }
  },
  methods: {
    onScreenshot: async function() {
      const canvas = this.$refs.canvas as HTMLCanvasElement;
      const video = this.$refs.video as HTMLVideoElement;
      const blob = await this.imageCapture.takePhoto();
      const image = await createImageBitmap(blob);
      if (canvas) {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const context2d = canvas.getContext("2d");
        if (context2d) {
          context2d.drawImage(image, 0, 0);
          this.showVideo = false;
          const model = await mobilenet.load();
          const predictions = await model.classify(canvas);
          console.log(predictions);
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
