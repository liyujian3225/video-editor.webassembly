<template>
  <div class="video-editor-container">
    <!-- 顶部按钮 -->
    <section class="controller-section">
      <div class="button-container">
        <div class="upload-btn-container">
          <input
            class="upload-input"
            type="file"
            accept="video/*"
            ref="videoInputElement"
            @change="addVideoOnCurrentSection($event)"
          />
          <el-button>上传视频</el-button>
        </div>
        <el-button @click="clearVideo">清空视频</el-button>
      </div>

      <div class="button-container">
        <el-button @click="drawer = true">视频当前参数</el-button>
      </div>

      <div class="button-container">
        <div class="upload-btn-container">
          <input
            class="upload-input"
            type="file"
            accept="audio/*"
            ref="audioInputElement"
            @change="addAudioOnCurrentSection($event)"
          />
          <el-button>上传配音</el-button>
        </div>
        <el-button @click="clearAudio">清空配音</el-button>
      </div>
    </section>

    <!-- 中部视频预览器 -->
    <video-player></video-player>

    <!-- 底部时间轴 -->
    <time-line></time-line>

    <el-drawer
      v-model="drawer"
      title="视频相关信息"
      :with-header="false">
      <div class="screenInfo">
        <div class="title">屏幕信息</div>
        <div>时间轴容器宽度 timeLineContainer_width：{{ timeLineContainer_width }}</div>
        <div>时间轴宽度 timeLine_width：{{ timeLine_width }}</div>
        <div>时间刻度尺宽度 timescale_width：{{ timescale_width }}</div>
        <div>时间轴左偏移量 timeLineOffsetLeft：{{ timeLineOffsetLeft }}</div>

        <div class="title">时间轴刻度信息</div>
        <div>格子宽度 gridWidth：{{ gridWidth }}</div>
        <div>格子内帧数 gridFrame：{{ gridFrame }}</div>
        <div>每组格子内帧数 groupGridFrame：{{ groupGridFrame }}</div>

        <div class="title">帧宽度信息</div>
        <div>当前帧宽度 frameWidth：{{ frameWidth }}</div>
        <div>合适帧宽度 fitFrameWidth：{{ fitFrameWidth }}</div>
        <div>最小帧宽度 frameWidth：{{ minFrameWidth }}</div>
        <div>最大帧宽度 frameWidth：{{ maxFrameWidth }}</div>
        <div>素材的最大帧数 maxFrameOfMaterial：{{ maxFrameOfMaterial }}</div>
      </div>
    </el-drawer>

  </div>

</template>

<script setup>
import VideoPlayer from "@/components/VideoPlayer.vue";
import TimeLine from "@/components/TimeLine.vue";
import Store from "@/store";
import { inject, computed, ref, onMounted } from "vue";
import Mapping from "@/map";
import Api from "@/api";
import { VideoEditor } from "@/viewmodels";
import WASM from "@/wasm";

const drawer = ref(false)

// 核心数据
const coreData = inject(Store.coreData);

// 时间轴的宽度
const timeLineContainer_width = inject(Store.timeLineContainer_width);

// 时间轴滚动轴左偏移量
const timeLineOffsetLeft = inject(Store.timeLineOffsetLeft);

// 帧宽度：决定了时间轴的比例
const frameWidth = inject(Store.frameWidth);

// 当前格子宽度
const gridWidth = inject(Store.gridWidth);

// 格子内帧数
const gridFrame = inject(Store.gridFrame);

// 每组格子内的帧数
const groupGridFrame = inject(Store.groupGridFrame);

// 时间轴宽度：用户能看见的宽度
const timeLine_width = inject(Store.timeLine_width);

// 时间刻度总宽度：包含用户看不见的宽度
const timescale_width = inject(Store.timescale_width);

// 当前预览器加载的视频 URL
const currentVideoUrl = inject(Store.currentVideoUrl);

// 素材的最大帧数
const maxFrameOfMaterial = inject(Store.maxFrameOfMaterial);

// 最大帧宽度
const maxFrameWidth = inject(Store.maxFrameWidth);

// 最小帧宽度
const minFrameWidth = inject(Store.minFrameWidth);

// 合适的帧宽度
const fitFrameWidth = inject(Store.fitFrameWidth);

// 当前段落级焦点
const currentSectionIndex = inject(Store.currentSectionIndex);

// 视频选择器 input type=file
const videoInputElement = ref(null);

// 读帧的 Worker
const readFrameWorker = inject(Store.readFrameWorker);

// 临时存放视频帧的列表
const videoFrameList = inject(Store.videoFrameList);

const currentFile = inject(Store.currentFile);

// 视频上传 Callback
const addVideoOnCurrentSection = (e) => {
  VideoEditor.addVideoOnCurrentSection(
    e.target.files,
    currentVideoUrl,
    coreData,
    frameWidth,
    fitFrameWidth,
    currentSectionIndex,
    videoInputElement
  );

  currentFile.value = e.target.files[0];

  // WASM.readFrame(
  //   readFrameWorker.value,
  //   e.target.files[0],
  //   49,
  //   52,
  //   "0",
  //   (imageData) => {
  //     // console.log("imageData", imageData);
  //   }
  // );
};

// 清空当前预览器的视频
const clearVideo = () => {
  VideoEditor.clearVideoOfCurrentSection(
    currentVideoUrl,
    coreData,
    currentSectionIndex,
    frameWidth,
    fitFrameWidth
  );
};

let audioObj = null;
const addAudioOnCurrentSection = (e) => {
  const file = e.target.files[0];
  const src = URL.createObjectURL(file)
  audioObj = new Audio(src);
  audioObj.addEventListener("canplaythrough", (event) => {
    /* 音频可以播放；如果权限允许则播放 */
    audioObj.play();
  });
}

const clearAudio = () => {
  if (audioObj) {
    audioObj.src = null;
  }
}

// 初始化：时间轴组件的宽度
timeLineContainer_width.value = Mapping.calcTimeLineContainerWidth(
  document.body.clientWidth
);

// 动态监听：窗口变化 -> 时间轴组件的宽度
window.onresize = () =>
  (() => {
    timeLineContainer_width.value = Mapping.calcTimeLineContainerWidth(
      document.body.clientWidth
    );
  })();
</script>

<style lang="scss" scoped>

.video-editor-container {
  width: calc(100% - 40px);
  height: calc(100% - 40px);
  border-radius: 20px;
  background: #393f4a;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: absolute;
  top: 0; bottom: 0;
  left: 0; right: 0;
  margin: auto;
}

.screenInfo {
  div {
    font-size: 20px;
    &.title {
      font-size: 24px;
      font-weight: bold;
      margin: 20px 0;
    }
  }
}

.controller-section {
  height: 80px;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.button-container {
  width: 200px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;

  .upload-btn-container {
    position: relative;
    width: 88px;
    height: 32px;
    background: #222222;
    color: #efefef;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    cursor: pointer;
    .upload-input {
      width: 100%;
      height: 100%;
      opacity: 0;
      position: absolute;
      cursor: pointer;
    }
  }
}
</style>
