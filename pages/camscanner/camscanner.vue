<template>
  <view>
		<web-view @message="onMessage" src="https://tony-xlh.github.io/mobile-document-scanning/mobile-web-capture-for-mini-program.html"></web-view>
	</view>
</template>

<script setup lang="ts">
  const onMessage = (e:any) => {
    console.log(e);
    savePDF(e.target.data[0].pdf);
  }
  const removeDataURLHead = (dataURL:string) => {
    return dataURL.substring(dataURL.indexOf(",")+1,dataURL.length);
  }
  
  const savePDF = (dataURL:string) => {
    console.log("share");
    const path = `${wx.env.USER_DATA_PATH}/scanned.pdf`;
    const fsm = wx.getFileSystemManager();
    fsm.writeFile({
      filePath: path,
      data: removeDataURLHead(dataURL),
      encoding: 'base64',
      success: () => {
         wx.openDocument({
          filePath: path,
          showMenu: true,
          success: () => {
            console.log('打开文档成功')
          },
          fail: () => {
            uni.showToast({
            	title: '打开文档失败。',
            	duration: 2000
            });
          }
        })
      },
      fail: () => {
        uni.showToast({
        	title: '保存文档失败。',
        	duration: 2000
        });
      }
    })
  }
</script>

<style>

</style>
