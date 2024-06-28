<template>
	<view class="container">
		<view class="intro">
      <button type="default" @click="goto('/pages/documentscanner/documentscanner')">前往文档扫描页面</button>
      <button type="default" @click="goto('/pages/camscanner/camscanner')">前往摄像头文档扫描页面</button>
    </view>
	</view>
</template>

<script setup lang="ts">
	const goto = (url:string) => {
    uni.navigateTo({
      url:url
    })
  }
</script>

<style>
	.container {
		padding: 20px;
		font-size: 14px;
		line-height: 24px;
	}
</style>
