<script>
import { makeIndividualStore } from '../stores/individual';

export default{
  name: "Camera",
  data() {
    return {
      height: 0,
      width: 320,
      streaming: false,
      toggleToSnap: false,
      video: undefined,
      canvas: undefined,
      picture: undefined,
      snapButton: undefined,
    }
  },
  computed: {
    data() {
      return makeIndividualStore(this.$route.params.name)()
    },
    source() {
      if(this.$route.params.type)
        return this.data[this.$route.params.type[0] + 'Img']
      // else
      //   return this.data.currentPicture
    },
  },
  mounted() {
    this.getElements()

    navigator.mediaDevices.getUserMedia({video: {facingMode: {exact: "environment"}}})
    .then((stream) => {
      this.video.srcObject = stream
      this.video.play()
    })
    .catch((error) => {
      console.log("ERROR: "+ error);
    })

    this.video.addEventListener(
      "canplay",
      () => {
        if (!this.streaming) {
          this.height = (this.video.videoHeight / this.video.videoWidth) * this.width;
        
          this.video.setAttribute("width", this.width);
          this.video.setAttribute("height", this.height);
          this.canvas.setAttribute("width", this.width);
          this.canvas.setAttribute("height", this.height);
          this.streaming = true;
        }
      },
      {once: true},
    );
    this.snapButton.addEventListener(
      "click",
      (ev) => {
        ev.preventDefault();
      },
      false
    );
  },
  methods: {
    getElements() {
      this.video = document.getElementById("video")
      this.canvas = document.getElementById("canvas")
      this.picture = document.getElementById('picture');
      this.snapButton = document.getElementById('snapButton');
    },
    takePicture() {
      const context = this.canvas.getContext("2d");
      if (this.width && this.height) {
        this.canvas.width = this.width;
        this.canvas.height = this.height;
        context.drawImage(this.video, 0, 0, this.width, this.height);
      
        const data = this.canvas.toDataURL("image/png");
        //this.data.currentPicture = data
        this.source.img = data
        this.data.ocr(data)
      } else {
        this.clearPicture();
      }
    },
    clearPicture() {
      const context = this.canvas.getContext("2d");
      context.fillStyle = "#AAA";
      context.fillRect(0, 0, this.canvas.width, this.canvas.height);

      const data = this.canvas.toDataURL("image/png");
      this.picture.setAttribute("src", data);
    },
  },
}
</script>

<template>
  <div id="parent">
    <canvas id="canvas"></canvas>
    <video id="video"></video>
    <Transition name="image"><img id="picture" :key="toggleToSnap" :src="this.source.img" v-if="this.source.img" /></Transition>
    <button id="backButton" @click="$router.push('/')">></button>
    <button id="snapButton" @click="takePicture(); toggleToSnap=!toggleToSnap;">Take picture</button>
  </div>
</template>

<style scoped>
#parent {
  /* display: flex;
  justify-content: center;
  align-items: center; */
  /* position: fixed; */
  height: 100vh;
  height: 100dvh;
  /* outline: 5px dashed red;
  outline-offset: -5px; */
}

#video {
  position: fixed;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  /* outline: 3px dashed blue; */
  outline-offset: -3px;
}

#picture {
  position: fixed;
  width: 30%;
  height: fit-content;
  /* height: 100dvh; */
  outline: 5px dashed black;
  outline-offset: -5px;
}

.image-enter-from {
  transform: scale(calc(20/3));
}

.image-enter-active {
  transition:  0.1s ease-out;
}

#snapButton {
  position: fixed;
  width: 10vh;
  height: 10vh;
  bottom: 10vh;
  left: calc(50vw - 5vh);
  border-radius: 50px;
  /* outline: 3px dashed orange; */
}

#backButton {
  float: right;
  font-size: 18px;
  height: 30px;
  width: 30px;
  margin: 10px;
  opacity: 0.8;
  border: none;
  background-color: transparent;
}

#canvas {
  display: none;
}
</style>