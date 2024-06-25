<template>
  <view>
    <view class="scanner">
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
      <button style="width:100%" type="default" @click="scan()">扫描</button>
      <view class="image-list">
        <view v-for="(scanned, index) in scans" class="scanned" >
          <image v-on:click="showAction(index);" mode="aspectFit" style="width: 100%; height: 200px;"  :src="scanned" alt=""/>
        </view>  
      </view>      
    </view>
    <view>
      <uni-popup ref="popup" type="dialog">
        <uni-popup-dialog class="popup-content" type="info" mode="base" content="保存该图？" :duration="0" :before-close="true" @close="closeDialog" @confirm="confirmDialog"></uni-popup-dialog>
      </uni-popup>
    </view>
    <view>
      <uni-popup ref="message" type="message">
        <uni-popup-message type="success" message="保存成功" :duration="2000"></uni-popup-message>
      </uni-popup>
    </view>
    <view v-if="status" class="mask">
      <text class="status">{{ status }}</text>
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
  const status = ref("");
  const popup = ref<any>();
  const message = ref<any>();
  const getScanners = () => {
    status.value = "读取扫描仪中列表……";
    uni.request({
        url: host+'DWTAPI/Scanners',
        data: {},
        header: {},
        success: (res) => {
          status.value = "";
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
          status.value = "done";
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
  let selectedScanIndex = -1;
  const scans = ref([]);
  const createScanJob = () => {
    status.value = "扫描中……";
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
          status.value = "";
        }
    });
  }
  
  const getDocumentImage = (jobID:string) => {
    status.value = "获取图片中……";
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
          const scanned = dataURL || ''
          scans.value.push(scanned);
          status.value = "";
        },
        fail: (res) => {
          console.log(res);
          status.value = "";
        }
    });
  }
  const scan = () => {
    console.log(selectedScanner);
    createScanJob();
  }
  
  const showAction = (index:number) => {
    console.log("clicked");
    selectedScanIndex = index;
    popup.value.open('top');
  }
  
  const closeDialog = () => {
    popup.value.close()
  }

  const confirmDialog = () => {
    console.log("share");
    const dataURL = scans.value[selectedScanIndex];
    const path = `${wx.env.USER_DATA_PATH}/${selectedScanIndex}.png`;
    const fsm = wx.getFileSystemManager();
    fsm.writeFile({
      filePath: path,
      data: dataURL.replace(/^data:image\/\w+;base64,/, ''),
      encoding: 'base64',
      success: () => {
        uni.saveImageToPhotosAlbum({
          filePath: path,
          success: () => {
            message.value.open();
          }
        });
      }
    })
    popup.value.close()
  }
</script>

<style scoped>
.scanned {
  margin-top: 10px;
}

.scanner {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.image-list {
  flex: 1;
  overflow: auto;
}

.mask {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: black;
  opacity: 70%;
}

.status {
  color: white;
  text-shadow: 1px 1px 1px black;
}

.popup-content {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  padding: 15px;
  background-color: #fff;
}
</style>
