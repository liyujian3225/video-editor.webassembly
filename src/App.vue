<template>
  <el-watermark
    :font="font"
    :content="['视频剪辑', 'video editor']">
    <div class="routerView">
      <router-view></router-view>
    </div>
  </el-watermark>
</template>

<script setup>
import { reactive, watch } from 'vue'
import Store from "@/store";
Store.useProvider();

const font = reactive({
  color: 'rgba(255, 255, 255, .15)',
  fontWeight: 'normal',
  fontSize: 20,
})
</script>

<style>
* {
  padding: 0;
  margin: 0;
  list-style: none;
  text-decoration: none;
}
div.routerView {
  width: 100vw;
  height: 100vh;
  background: #ffffff;
  position: relative;
}
</style>
