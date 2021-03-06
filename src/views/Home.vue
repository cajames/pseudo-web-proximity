<template>
  <div class="container mx-auto text-white text-center h-screen flex flex-col justify-center pt-8">

    <!-- Warning -->
    <div class="bg-blue mx-4 rounded mb-8 p-4" v-if="!cameraAvailable">
      <span>The camera is required for this to work</span>
    </div>

    <!-- Text -->
    <div class="text-white mb-8 text-4xl">
      <span v-if="isCovered" class="block">Covered</span>
      <span v-else class="block">Not Covered</span>
      <span class="text-10xl font-bold">{{countChanges}}</span>
    </div>

    <!-- Camera Details -->
    <div v-show="showDetails" class="pt-2 bg-blue mx-4 p-4 rounded" >
      <div v-if="cameraAvailable" class="flex flex-col items-center">
        <video ref="video" width="100" height="100" playsinline autoplay></video>
        <canvas class="hidden" ref="canvas" width="100" height="100"></canvas>
        <span class="text-white mb-2">Brightness: {{brightness}}</span>
        <div class="flex mb-2">
          <label for="treshold" class="mr-2">Treshold:</label>
          <input type="number" name="treshold" v-model="treshold">
        </div>
        <div class="flex mb-2">
          <label for="refresh" class="mr-2">Refresh:</label>
          <input type="number" name="refresh" v-model="refresh">
        </div>
      </div>
      <span v-else class="text-xl">Camera not available.</span>
    </div>

    <!-- Footer -->
    <button @click="toggleDetails" class="fixed pin-b pin-x bg-blue p-4 w-screen shadow-md text-white">
      <span v-if="!showDetails">Show details</span>
      <span v-else>Hide details</span>
    </button>

  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import { setInterval, clearInterval } from "timers";
import debounce from "lodash/debounce";

@Component
export default class Home extends Vue {
  cameraAvailable: boolean = false;
  context: any = null;
  video: any = null;
  brightness: any = null;
  interval: any = null;
  countChanges: number = 0;
  showDetails: boolean = false;
  treshold: number = 25;
  refresh: number = 50;

  created() {
    this.setupVideoStream();
  }

  beforeDestroy() {
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  toggleDetails() {
    this.showDetails = !this.showDetails;
  }

  get isCovered() {
    if (this.brightness < this.treshold) {
      return true;
    }
    return false;
  }

  renderImageFromWebcam() {
    if (this.cameraAvailable && this.context !== null) {
      this.context.drawImage(this.video, 0, 0, 100, 100);
      this.calculateImageData();
    }
  }

  startIntervalScan() {
    this.interval = setInterval(() => {
      this.renderImageFromWebcam();
    }, this.refresh);
  }

  calculateImageData() {
    const imageData = this.context.getImageData(0, 0, 100, 100);
    const data = imageData.data;
    const length = data.length;
    let colorSum = 0;

    for (let x = 0; x < length; x += 4) {
      const r = data[x];
      const g = data[x + 1];
      const b = data[x + 2];
      const avg = Math.floor((r + g + b) / 3);
      colorSum += avg;
    }

    const brightness = Math.floor(colorSum / (100 * 100));
    this.brightness = brightness;
  }

  updateCoverCount(newVal: boolean, oldVal: boolean) {
    if (newVal === true && oldVal === false) {
      this.countChanges += 1;
    }
  }

  //#region promise video
  // async setupVideoStream() {
  //   try {
  //     if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {

  //       console.log('Video was allowed')
  //       // Video was allowed
  //       this.cameraAvailable = true;

  //       const stream = await navigator.mediaDevices.getUserMedia({
  //         video: {
  //           facingMode: "user"
  //         },
  //         audio: false
  //       });

  //       console.log('Got the stream')

  //       this.video = this.$refs["video"];
  //       const canvas: any = this.$refs["canvas"];
  //       this.context = canvas.getContext("2d");
  //       this.video.src = window.URL.createObjectURL(stream);

  //       console.log('Setup video')

  //       this.startIntervalScan();

  //       console.log('Started Scan')
  //     }
  //   } catch (error) {
  //     console.log("Failed to get camera access.");
  //     this.cameraAvailable = false;
  //   }
  // }
  //#endregion

  setupVideoStream() {
    navigator.getUserMedia =
      navigator.getUserMedia ||
      (navigator as any).webkitGetUserMedia ||
      (navigator as any).mozGetUserMedia;

    try {
      if (navigator.getUserMedia) {
        console.log("Video was allowed");
        // Video was allowed
        this.cameraAvailable = true;

        const videoContraints = {
          video: {
            facingMode: "user"
          },
          audio: false
        };

        navigator.getUserMedia(
          videoContraints,
          stream => {
            const video: any = this.$refs["video"];
            const canvas: any = this.$refs["canvas"];
            this.context = canvas.getContext("2d");
            this.video = video;
            video.srcObject = stream;
            this.startIntervalScan();
          },
          err => {
            throw err;
          }
        );
      }
    } catch (error) {
      console.log("Failed to get camera access.");
      this.cameraAvailable = false;
    }
  }

  @Watch("isCovered")
  onIsCoveredChanged(newVal: boolean, oldVal: boolean) {
    this.updateCoverCount(newVal, oldVal);
    // debounce(this.updateCoverCount, 100, { leading: true });
  }
}
</script>
