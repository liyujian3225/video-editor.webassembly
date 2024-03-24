<template>
  <div class="video-player-container">
    <video
      id="video"
      autoplay
      :volume="volume"
      :playbackRate="playbackRate"
      :src="currentVideoUrl ? currentVideoUrl : null"
    ></video>
    <div class="button-container">
      <span class="current-time">{{
        convertSecondsToHMS(currentDuration)
      }}</span>
      <span class="total-time">{{ convertSecondsToHMS(totalDuration) }}</span>
      <div class="play">
        <span @click="playOrPauseVideo">
          <el-icon v-if="isPlay"><VideoPause /></el-icon>
          <el-icon v-else><VideoPlay /></el-icon>
        </span>
        <el-icon @click="screenshot"><Picture /></el-icon>
      </div>
    </div>
    <div class="controlBar">
      <el-icon color="#ffffff">
        <Headset />
      </el-icon>
      <el-slider
        style="width: 200px; margin: 0 15px 0 10px"
        size="small"
        :min="0"
        :max="1"
        :show-stops="true"
        :show-tooltip="false"
        :step="0.1"
        v-model="volume"
      />
      <el-dropdown @command="handleCommand">
        <el-text style="cursor: pointer; color: #ffffff;" type="primary">倍速</el-text>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item command="0.5">0.5x</el-dropdown-item>
            <el-dropdown-item command="0.75">0.75x</el-dropdown-item>
            <el-dropdown-item command="1.0">1.0x</el-dropdown-item>
            <el-dropdown-item command="1.25">1.25x</el-dropdown-item>
            <el-dropdown-item command="1.5">1.5x</el-dropdown-item>
            <el-dropdown-item command="2.0">2.0x</el-dropdown-item>
            <el-dropdown-item command="3.0">3.0x</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<script setup>
import { inject, onMounted, ref, watch } from 'vue'
import Store from '@/store'

// 初始化视频URL
let video = null
const isPlay = ref(true)
const volume = ref(1)
const playbackRate = ref(1)
const currentVideoUrl = inject(Store.currentVideoUrl)
const totalDuration = ref(0) //秒
const currentDuration = ref(0) //秒
const timeLineOffsetLeft = inject(Store.timeLineOffsetLeft)
const timescale_width = inject(Store.timescale_width)

onMounted(() => {
  watchVideoProgress()
})
const watchVideoProgress = () => {
  video = document.getElementById('video')
  video.addEventListener('loadedmetadata', function () {
    totalDuration.value = video.duration
  })
  video.addEventListener('timeupdate', function () {
    const { currentTime } = video
    currentDuration.value = currentTime
  })
  video.addEventListener('play', function () {})
}

watch(timeLineOffsetLeft, function (v) {
  const radio = timeLineOffsetLeft.value / timescale_width.value
  video.currentTime = totalDuration.value * radio
  video.play()
})

const handleCommand = (command) => {
  playbackRate.value = Number(command)
}

const playOrPauseVideo = () => {
  if (video) {
    if (isPlay.value) {
      video.pause()
      isPlay.value = false
    } else {
      video.play()
      isPlay.value = true
    }
  }
}

const convertSecondsToHMS = (seconds) => {
  var hours = Math.floor(seconds / 3600)
  var minutes = Math.floor((seconds % 3600) / 60)
  var remainingSeconds = Math.floor(seconds % 60)
  hours = String(hours).padStart(2, '0')
  minutes = String(minutes).padStart(2, '0')
  remainingSeconds = String(remainingSeconds).padStart(2, '0')
  return hours + ':' + minutes + ':' + remainingSeconds
}

const screenshot = () => {
  const canvas = document.createElement('canvas')
  const a = document.createElement('a')
  if (video) {
    canvas.width = video.clientWidth
    canvas.height = video.clientHeight
    const ctx = canvas.getContext('2d')
    if (ctx && video.videoWidth && video.videoHeight) {
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height)
      const filename = 'file_' + new Date().getTime() + '.png'
      a.setAttribute('download', filename)
      a.href = canvas.toDataURL('image/png')
      a.click()
    }
  }
}
</script>

<style lang="scss" scope>
.video-player-container {
  width: 100%;
  height: 445px;
  display: flex;
  flex-direction: column;
  align-items: center;

  video {
    width: 658px;
    height: 370px;
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
      width: 60px;
      height: 16px;
      line-height: 16px;
      position: absolute;
      top: 0;
      bottom: 0;
      right: 0;
      left: 0;
      margin: auto;
      display: flex;
      justify-content: space-around;
      .el-icon {
        color: #ffffff;
        font-size: 16px;
        cursor: pointer;
      }
    }
  }

  .controlBar {
    width: 658.86px;
    display: flex;
    align-items: center;
    justify-self: start;
  }
}
</style>
