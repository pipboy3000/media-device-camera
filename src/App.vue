<template>
  <div id="app">
    <div>
      <video ref="video" autoplay v-if="showVideo" />
      <canvas ref="canvas" style="display: none" />
      <div>
        <button @click="onScreenshot" v-if="showVideo">撮影</button>
      </div>
    </div>
    <div>
      <img class="screenshot-image" v-bind:src="screenshotImage" />
      <a :href="screenshotImage" v-if="!showVideo" download="screenshot.jpeg"
        >ダウンロード</a
      >
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "app",
  data() {
    return {
      options: [] as MediaDeviceInfo[],
      selectedCamera: "",
      streamStarted: false,
      streamMedia: new MediaStream(),
      screenshotImage: "",
      showVideo: true,
      constraints: {
        video: {
          width: {
            min: 640,
            ideal: 1920,
            max: 2560
          },
          height: {
            min: 480,
            ideal: 1080,
            max: 1440
          }
        }
      }
    };
  },
  computed: {},
  async mounted() {
    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      this.selectedCamera = devices.filter(
        device => device.kind === "videoinput"
      )[0].deviceId;

      const constraints = {
        ...this.constraints,
        deviceId: {
          exact: this.selectedCamera
        }
      };

      this.streamMedia = await navigator.mediaDevices.getUserMedia(constraints);
      const video = this.$refs.video as HTMLVideoElement;
      video.srcObject = this.streamMedia;
      this.streamStarted = true;
    } catch (error) {
      this.streamStarted = false;
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
          this.screenshotImage = canvas.toDataURL("image/jpeg");
          this.showVideo = false;
        }
      }
    }
  }
});
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;

  > a {
    display: inline-block;
  }
}
</style>
