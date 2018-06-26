<template>
  <div class="container mx-auto">
	<div v-if="cameraAvailable">
		<video ref="video" width="100" height="100" autoplay></video>
		<canvas class="hidden" ref="canvas" width="100" height="100"></canvas>
	</div>
	<div v-else>
		<span class="text-xl">Camera not available.</span>
	</div>

	<div class="text-center">
		<span v-if="isCovered" class="block">Covered</span>
		<span v-else class="block">Not Covered</span>
		<span>{{countChanges}} changes</span>
	</div>
	<span>Brightness: {{brightness}}</span>

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

  created() {
    this.setupVideoStream();
  }

  beforeDestroy() {
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  get isCovered() {
    if (this.brightness < 25) {
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
    }, 50);
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

  incrementCount = debounce(
    () => {
      console.log("whaaaa", this, this.countChanges);
      debugger;
      this.countChanges = this.countChanges + 1;
    },
    100,
    { leading: true }
  );

  updateCoverCount(newVal: boolean, oldVal: boolean) {
    if (newVal === true && oldVal === false) {
      this.countChanges += 1;
    }
  }

  async setupVideoStream() {
    try {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        // Video was allowed
        this.cameraAvailable = true;

        const stream = await navigator.mediaDevices.getUserMedia({
          video: true
        });

        this.video = this.$refs["video"];
        const canvas: any = this.$refs["canvas"];
        this.context = canvas.getContext("2d");
        this.video.src = window.URL.createObjectURL(stream);

        this.startIntervalScan();
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
