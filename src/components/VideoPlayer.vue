<template>
  <div class="video-player-container">
    <video
      id="video"
      autoplay
      :src="currentVideoUrl ? currentVideoUrl : null">
    </video>
    <div class="button-container">
      <span class="current-time">{{ convertSecondsToHMS(currentDuration) }}</span>
      <span class="total-time">{{ convertSecondsToHMS(totalDuration) }}</span>

      <div class="play">播放</div>
    </div>
  </div>
</template>

<script setup>
import {inject, onMounted, ref, watch} from "vue";
import Store from "@/store";

// 初始化视频URL
let video = null;
const currentVideoUrl = inject(Store.currentVideoUrl);
const totalDuration = ref(0);  //秒
const currentDuration = ref(0);  //秒
const timeLineOffsetLeft = inject(Store.timeLineOffsetLeft);
const timescale_width = inject(Store.timescale_width);

onMounted(() => { watchVideoProgress()});
const watchVideoProgress = () => {
  video = document.getElementById("video");
  video.addEventListener('loadedmetadata', function() {
    totalDuration.value = video.duration;
  });
  video.addEventListener("timeupdate", function() {
    const { currentTime } = video;
    currentDuration.value = currentTime;
  })
  video.addEventListener("play", function() {

  })
}

watch(timeLineOffsetLeft, function(v) {
  const radio = timeLineOffsetLeft.value / timescale_width.value;
  video.currentTime = totalDuration.value * radio;
  video.play();
})

const convertSecondsToHMS = (seconds) => {
  var hours = Math.floor(seconds / 3600);
  var minutes = Math.floor((seconds % 3600) / 60);
  var remainingSeconds = Math.floor(seconds % 60);
  hours = String(hours).padStart(2, "0");
  minutes = String(minutes).padStart(2, "0");
  remainingSeconds = String(remainingSeconds).padStart(2, "0");
  return hours + ":" + minutes + ":" + remainingSeconds;
}

</script>

<style lang="scss" scope>
.video-player-container {
  width: 100%;
  height: 422px;
  display: flex;
  flex-direction: column;
  align-items: center;

  video {
    width: 658.86px;
    height: 370.61px;
    background: #000000;
    margin-top: 15px;
    margin-bottom: 10px;
  }

  .button-container {
    width: 658.86px;
    display: flex;
    justify-content: space-between;
    position: relative;

    .current-time {
      font-size: 12px;
      margin-right: 14px;
      color: #7f72e5;
    }

    .total-time {
      font-size: 12px;
      margin-right: 14px;
      color: #bbbbbb;
    }

    .play {
      position: absolute;
      top: 0;
      bottom: 0;
      right: 0;
      left: 0;
      margin: auto;
      width: 30px;
      text-align: center;
      font-size: 12px;
      font-weight: bold;
      color: #efefef;
      cursor: pointer;
    }
  }
}
</style>
