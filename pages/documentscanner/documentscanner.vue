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
        v-model="selectedColorMode"
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

      <view>
        <image mode="aspectFit" style="width: 100%; height: 200px;"  :src="scanned" alt=""/>
      </view>

  </view>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
  const host = "http://192.168.8.65:18622/"
  const license = "t01908AUAAK4ItCagsC9OlLgF9scuuocRrXPNxV2DWKx3GZVuH5zbgbPPmuwLI9LBt/YM72qF6Tp2VnV45o7gjBFrA4vrVQrXLyc7OLW9U6W9Ex2cfOSUNF4irKftlnllAjPwXgDdr8MOoAbWXE7AZ9hygwXQAtQA1MoBFnC7iuvmU84FUv/950CLkx2c2t5ZF0gbJzo4+ciZCsQHcVNa7bQWCOqbcwBoAXoLYHvILgUiZ4AWoBcAMT3y6mUGqVcq8w==";
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
  let selectedColorMode = 0;
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
  const scanned = ref("");
  const createScanJob = () => {
    uni.request({
        url: host+'DWTAPI/ScanJobs',
        data: {
          "license":license,
          "device":selectedScanner,
          "config":{ //Device configuration https://www.dynamsoft.com/web-twain/docs/info/api/Interfaces.html#DeviceConfiguration (optional)
             "IfShowUI":false, //show the UI of the scanner
             "Resolution":selectedResolution,
             "IfFeederEnabled":false, //enable auto document feeder
             "IfDuplexEnabled":false //enable duplex document scanning
           },
           "caps":{ // Capabilities https://www.dynamsoft.com/web-twain/docs/info/api/Interfaces.html#capabilities (optional)
             "exception":"ignore",
             "capabilities":[
               {
                 "capability":257, //pixel type
                 "curValue":selectedColorMode //0: black&white, 1: gray, 2: color
               }
             ]
           }
        },
        method:"POST",
        header: {},
        success: (res) => {
          console.log(res);
          getDocumentImage(res.data as string)
        },
        fail: (res) => {
          console.log(res);
        }
    });
  }
  
  const getDocumentImage = (jobID:string) => {
    uni.request({
        url: host+"DWTAPI/ScanJobs/"+jobID+"/NextDocument",
        data: {},
        responseType: 'arrayBuffer',
        method:"GET",
        header: {},
        success: (res) => {
          console.log(res);
          const arrayBuffer = new Uint8Array(res.data as ArrayBuffer)
          const dataURL = "data:image/png;base64," + uni.arrayBufferToBase64(arrayBuffer)
          scanned.value = dataURL || ''
        },
        fail: (res) => {
          console.log(res);
        }
    });
  }
  const scan = () => {
    console.log(selectedScanner);
    createScanJob();
  }
</script>

<style>

</style>
