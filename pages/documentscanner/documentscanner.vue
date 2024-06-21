<template>
  <view>
    <uni-section title="扫描仪:" type="line">
      <uni-data-select
        v-model="selectedScanner"
        :localdata="scanners"
      ></uni-data-select>
    </uni-section>
    <button type="default" @click="scan()">扫描</button>
  </view>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
  const host = "http://192.168.8.65:18622/"
  onMounted(() => {
    getScanners();
  })
  const getScanners = () => {
    uni.request({
        url: host+'DWTAPI/Scanners',
        data: {},
        header: {},
        success: (res) => {
          console.log(res.data);
          let devices:any[] = res.data as any[];
          let connectedScanners = [];
          for (var i = 0; i < devices.length; i++) {
            let device = devices[i];
            let connectedScanner = {
              value:device.device,
              text:device.name
            }
            connectedScanners.push(connectedScanner);
          }
          scanners.value = connectedScanners;
        },
        fail: (res) => {
          console.log(res);
        }
    });
  }
  
  let selectedScanner = -1;
  let scanners = ref([]);
  const scan = () => {
    console.log(selectedScanner);
  }
</script>

<style>

</style>
