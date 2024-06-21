<template>
  <view>
    <uni-section title="扫描仪:" type="line">
      <uni-data-select
        v-model="selectedScanner"
        :localdata="scanners"
      ></uni-data-select>
    </uni-section>
    <uni-section title="颜色模式:" type="line">
      <uni-data-select
        v-model="selectedColorModes"
        :localdata="colorModes"
      ></uni-data-select>
    </uni-section>
    <uni-section title="分辨率:" type="line">
      <uni-data-select
        v-model="selectedResolution"
        :localdata="resolutions"
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
  let selectedColorModes = 0;
  let colorModes = [
    {
      value:0,text:"黑白"
    },{
      value:1,text:"灰度"
    },{
      value:2,text:"彩色"
    }
  ];
  let selectedResolution = 200;
  let resolutions = [
    {
      value:300,text:"300"
    },{
      value:200,text:"200"
    },{
      value:100,text:"100"
    }
  ]
  const scan = () => {
    console.log(selectedScanner);
  }
</script>

<style>

</style>
