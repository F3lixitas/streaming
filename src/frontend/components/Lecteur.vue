<template>
  <div id="container" class="container" @mouseover="over" @mousemove="move" @mouseleave="mouseExit">
    <video id="video" width="1280" height="720">
      <source @src="source_file" type="video/mp4"/>
    </video>
    <div id="controls" class="controls" @mouseover="mouseOverControls" @mouseleave="mouseExitsControls">
      <div style="position: absolute; top: 5px; width: 100%; height: 40px;">
        <button class="" @click="play" style="position: absolute; left: 0; border-radius: 5px"><i id="play" class="material-icons" style="font-size: 40px; color: rgba(250, 50, 100, 1)">play_arrow</i></button>
        <button class="" @click="mute" style="position: absolute; left: 50px; border-radius: 5px"><i id="sound" class="material-icons" style=" font-size: 30px; color: rgba(250, 50, 100, 1)">volume_up</i></button>
        <input @input="volumeChange" type="range" min="0" max="100" value="100" class="slider" id="soundSlider" style="position: absolute; left: 110px; top: 0; width: 100px; height: 35px; color: rgba(250, 50, 100, 1)"/>
        <button class="" @click="switchFullscreen" style="position: absolute; right: 0; border-radius: 5px"><i id="fullscreen" class="material-icons" style="font-size: 40px; color: rgba(250, 50, 100, 1)">fullscreen</i></button>
        <button id="settingsButton" class="" @click="openSettings" style="position: absolute; right: 50px; border-radius: 5px"><i id="fullscreen" class="material-icons" style="font-size: 30px; padding-top: 2px; color: rgba(250, 50, 100, 1)">settings</i></button>
        <span style="position: absolute; display: flex; justify-content: space-between; border-radius: 5px; padding: 6px 5px; right: 100px; width: 100px; top: 5px; bottom: 5px; background-color: rgba(200, 100, 100, 0.4)"><div id="currentTime" style="margin: 0px 5px">0:0</div> / <div id="totalTime" style="margin: 0px 5px">0:0</div></span>
      </div>
      <div style="position: absolute; width: 100%; height: 5px; top: 0px">
        <progress id="videoProgress" max="100" value="0" style="position: absolute; top: 0px; border: none; width: 100%; height: 5px;"></progress>
      </div>
    </div>
    <div style="position: absolute; top: 0; bottom: 45px; width: 100%" @click="play"></div>
    <div id="settingsPanel" style="position: absolute; visibility: hidden; bottom: 50px; right: 5px; width: 200px; height: 300px; background-color: rgba(0, 0, 0, 0.15); backdrop-filter: blur(10px); border-radius: 10px">

    </div>
  </div>

</template>

<script>
export default {
  name: "Lecteur",
  setup() {
    return {
      source_file : String("data/24hcinderella.mp4"),
      playing : Boolean(false),
      fullscreen : Boolean(false),
      moved : Number(0),
      mouseOver : Boolean(false),
      mouseOverC : Boolean(false),
      muted : Boolean(false),
      previousVolume : Number(1),
      settingsOpened : Boolean(false)
    }
  },
  mounted(){
    let container = document.getElementById("container");
    let video = document.getElementById("video");
    video.children[0].setAttribute("src", this.source_file);
    let videoProgress = document.getElementById("videoProgress");
    video.addEventListener('loadedmetadata', () => {
      videoProgress.setAttribute("max", video.duration);
      let totalTime = document.getElementById("totalTime");
      totalTime.innerHTML = this.toTime(video.duration);
    });
    video.addEventListener('timeupdate', () =>{
      videoProgress.value = video.currentTime;
      let currentTime = document.getElementById("currentTime");
      currentTime.innerHTML = this.toTime(video.currentTime);
    });
    video.addEventListener('ended', () => {
      let playButton = document.getElementById("play");
      playButton.innerHTML = "play_arrow";
      this.playing = false;
    });

    let mouseDown = false;
    let flaggedForClosure = false;

    videoProgress.addEventListener('mouseenter', (e)=>{
      flaggedForClosure = false;
      videoProgress.className = "progressHover";
      videoProgress.style.height = '10px';
      videoProgress.style.top = '0px';
    });
    videoProgress.addEventListener('mouseleave', (e)=>{
      if(mouseDown) {
        flaggedForClosure = true;
        return;
      }
      videoProgress.className = "";
      videoProgress.style.height = '5px';
    });

    videoProgress.addEventListener('mousedown', (e)=> {
      const rect = videoProgress.getBoundingClientRect();
      const pos = (e.pageX  - rect.left) / videoProgress.offsetWidth;
      video.currentTime = pos * video.duration;
      mouseDown = true;
    });

    container.addEventListener('mousemove', (e)=> {
      if(mouseDown){
        const rect = videoProgress.getBoundingClientRect();
        const pos = (e.pageX  - rect.left) / videoProgress.offsetWidth;
        video.currentTime = pos * video.duration;
      }
    });

    const containsMouse = (mouse, rect) => {
      console.log("pageX : ", mouse.pageX, "\npageY : ", mouse.pageY);
      console.log(rect);
      return (mouse.pageX > rect.left) && (mouse.pageX < rect.right) && (mouse.pageY > rect.top) && (mouse.pageY < rect.bottom);

    }

    let settingsPanel = document.getElementById("settingsPanel");
    let settingsButton = document.getElementById("settingsButton");
    window.addEventListener('click', (e)=> {
      
      if((!containsMouse(e, settingsPanel.getBoundingClientRect()) && !containsMouse(e, settingsButton.getBoundingClientRect())) && this.settingsOpened){
        this.openSettings();
      }
    })

    container.addEventListener('mouseup', (e)=> {
      mouseDown = false;
      if(flaggedForClosure){
        videoProgress.className = "";
        videoProgress.style.height = '5px';
      }
    });
  },
  methods: {
    toTime(seconds){
      const hours = Math.floor(seconds/3600);
      const minutes = Math.floor((seconds-hours*3600)/60);

      if(hours>0){
        return hours + ":" + minutes + ":" + Math.floor(seconds-hours*3600-minutes*60);
      } else{
        return minutes + ":" + Math.floor(seconds-minutes*60);
      }
    },
    volumeChange(){
      let video = document.getElementById("video");
      let soundSlider = document.getElementById("soundSlider");
      video.volume = soundSlider.value/100;
    },
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
    openSettings(){
      let settingsPanel = document.getElementById("settingsPanel");
      if(this.settingsOpened){
        settingsPanel.style.visibility = "hidden";
      } else {
        settingsPanel.style.visibility = "visible";
      }
      this.settingsOpened = !this.settingsOpened;
    },
    mute(){
      let video = document.getElementById("video");
      let soundSlider = document.getElementById("soundSlider");
      let soundButton = document.getElementById("sound");
      if(this.muted){
        soundSlider.value = this.previousVolume*100;
        video.volume = this.previousVolume;
        soundButton.innerHTML = "volume_up";
      } else {
        this.previousVolume = soundSlider.value/100;
        soundSlider.value = 0;
        video.volume = 0;
        soundButton.innerHTML = "volume_off";
      }
      this.muted = !this.muted;
    },
    switchFullscreen(){
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
    over(){
      this.mouseOver = true;
    },
    hover(){
      let controls = document.getElementById("controls");
      controls.className = "controls enterControls";
      controls.style.height = "45px";
    },
    move(){
      let container = document.getElementById("container");
      container.style.cursor = "default";
      this.hover();
      this.moved += 1;
      setTimeout(() => {
        this.moved -= 1;
        if(this.moved <= 0 && !this.mouseOverC){
          this.unhover();
          if(this.mouseOver){
            container.style.cursor = "none";
          }
        }
      }, 1000);
    },
    mouseExit(){
      this.mouseOver = false;
      this.unhover();
    },
    unhover(){
      let controls = document.getElementById("controls");
      controls.className = "controls exitControls";
      controls.style.height = "0px";
    },
    mouseOverControls(){
      this.mouseOverC = true;
    },
    mouseExitsControls(){
      this.mouseOverC = false;
    }
  }
}
</script>

<style scoped>
progress{
  color: rgba(250, 50, 100, 1);
  background-color: rgba(0, 0, 0, 0.5);
}
progress::-moz-progress-bar{
  background-color: rgba(250, 50, 100, 1);
}
progress::-webkit-progress-value{
  background-color: rgba(250, 50, 100, 1);
}
.slider{
  background-color: rgba(0, 0, 0, 0);
}
.slider::-moz-range-progress{
  background-color: rgba(250, 50, 100, 1);
  height: 5px;
}
.slider::-moz-range-track{
  background-color: rgba(0, 0, 0, 0.5);
  height: 5px;
}
.slider::-moz-range-thumb{
  background-color: rgba(50, 50, 50, 1);
  border: none;
}
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
  animation-duration: 0.1s;
}
.exitControls{
  overflow: hidden;
  animation-name: exit;
  animation-duration: 0.1s;
}
.progressHover{
  animation-name: progress;
  animation-duration: 0.1s;
}
@keyframes progress {
  0% {
    height: 5px;
  }
  100% {
    height: 10px;
  }
}
@keyframes enter {
  0% {
    height: 0px;
    max-height: 0px;
    opacity: 0%;
  }
  100%{
    height: 45px;
    max-height: 45px;
    opacity: 100%;
  }
}
@keyframes exit {
  0% {
    height: 45px;
    max-height: 45px;
    opacity: 100%;
  }
  100%{
    height: 0px;
    max-height: 0px;
    opacity: 0%;
  }
}
</style>