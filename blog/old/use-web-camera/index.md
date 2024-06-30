---
date: 2021-04-20
---

# 浏览器如何调用用户摄像头

目前一个需求：要求用户开启摄像头进行人脸验证，验证完成后再关闭摄像头进行提示。如何进行人脸识别，只需要上传视频到服务器就行，考虑处理如何使用用户摄像头并进行实时预览。

这里主要是使用 MediaStream 获取视频流，然后利用 video 标签显示视频获取内容。Javascript 代码如下：

<!--- truncate -->

```js
//获取video节点
let video = document.getElementById("video") 
const config = {
  //指定摄像头画面尺寸，单位像素，也可以不指定具体大小，使用true表示使用
  video:{ width:150, height:150 },  
  //不使用音频
  audio:false
}
/*getUserMedia会返回一个Promise，如果用户允许使用，那么resolve回调MediaStream，否则reject抛出异常*/
navigator.mediaDevices.getUserMedia(config)  
  .then(MediaStream => {
    video.srcObject = MediaStream
    video.play()  //指定视频源并播放
  
  // 这里模拟识别成功，关闭摄像头，需要在MediaStream的Track里关闭掉我们使用的Track
    setTimeout(()=>{
      MediaStream.getVideoTracks()[0].stop()  //只会有一个视频流
      MediaStream.removeTrack(MediaStream.getVideoTracks()[0])
      video.srcObject = null  //srcObject恢复默认属性
    },4000)
  })
```

需要注意的是，目前很多文章只讲了如何调用摄像头而不讲如何关闭，或者关闭的方法也只有 removeTrack 甚至只有关闭 video，这种几乎是掩耳盗铃的行为，因为这样做只是断开了媒体流，硬件依然在工作。所以使用 `MediaStream.getTracks()[].stop()` 来关闭硬件。getTrack 返回所使用的 Track 列表，因此需要逐个关闭：
```js
MediaStream.getTracks().forEach(track=>{
    track.stop()
  }
)
```
  
  
> 参考资料  
[MediaDevices.getUserMedia() - Web API | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices/getUserMedia)  
[媒体流 (MediaStream) - Web API | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaStream)