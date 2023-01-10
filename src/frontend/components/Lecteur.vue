<template>
  <div id="container" class="container" @mouseover="hover" @mouseleave="unhover">
    <video id="video" width="1280" height="720">
      <source src="data/24hcinderella.mp4" type="video/mp4"/>
    </video>
    <div id="controls" class="controls">
      <div style="position: relative; width: 100%; height: 5px">
        <progress max="100" value="20" style="position: absolute; top: 0; border: none; width: 100%; height: 5px; color: rgba(250, 50, 100, 1)"></progress>
      </div>
      <div style="position: relative; width: 100%; height: 40px;">
        <button class="" @click="play" style="position: absolute; left: 0; border-radius: 5px"><i id="play" class="material-icons" style="font-size: 40px; color: rgba(250, 50, 100, 1)">play_arrow</i></button>
        <i id="sound" class="material-icons" style="position: absolute; left: 60px; top: 6px; font-size: 30px; color: rgba(250, 50, 100, 1)">volume_up</i><input type="range" min="0" max="100" value="100" class="slider" id="soundSlider" style="position: absolute; left: 100px; top: 0; width: 100px; height: 40px"/>
        <button class="" @click="switchFullscreen" style="position: absolute; right: 0; border-radius: 5px"><i id="fullscreen" class="material-icons" style="font-size: 40px; color: rgba(250, 50, 100, 1)">fullscreen</i></button>
      </div>
    </div>
  </div>

</template>

<script>
export default {
  name: "Lecteur",
  setup() {
    return {
      playIcon : String("play_arrow"),
      playing : Boolean(false),
      fullscreen : Boolean(false)
    }
  },
  methods: {
    play(){
      let video = document.getElementById("video");
      let playButton = document.getElementById("play");
      if(this.playing){
        playButton.innerHTML = "play_arrow";
        video.pause();
      } else {
        playButton.innerHTML = "pause";
        video.play();
      }
      this.playing = !this.playing;
    },
    switchFullscreen(){
      let video = document.getElementById("video");
      let container = document.getElementById("container");
      let fullscreenButton = document.getElementById("fullscreen");
      if(this.fullscreen){
        fullscreenButton.innerHTML = "fullscreen";
        document.exitFullscreen();
      } else {
        fullscreenButton.innerHTML = "fullscreen_exit";
        container.requestFullscreen();
      }
      this.fullscreen = !this.fullscreen;
    },
    hover(){
      let controls = document.getElementById("controls");
      controls.className = "controls enterControls";
      controls.style.height = "40px";
    },
    unhover(){
      let controls = document.getElementById("controls");
      controls.className = "controls exitControls";
      controls.style.height = "0px";
    }
  }
}
</script>

<style scoped>
video{
  height: 100%;
  width: 100%;
}
button{
  height: 40px;
  width: 50px;
  border: none;
  background-color: rgba(0, 0, 0, 0.0);
}
button:hover{
  background-color: rgba(100, 100, 100, 0.3);
}
.container{
  position: relative;
}
.controls{
  text-align: left;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 45px;
  background: rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(10px);
}
.enterControls{
  overflow: hidden;
  animation-name: enter;
  animation-duration: 0.2s;
}
.exitControls{
  overflow: hidden;
  animation-name: exit;
  animation-duration: 0.2s;
}
@keyframes enter {
  0% {
    height: 0px;
    max-height: 0px;
  }
  100%{
    height: 45px;
    max-height: 45px;
  }
}
@keyframes exit {
  0% {
    height: 45px;
    max-height: 45px;
  }
  100%{
    height: 0px;
    max-height: 0px;
  }
}
</style>