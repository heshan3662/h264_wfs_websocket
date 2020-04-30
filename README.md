# h264_wfs_websocket
html5 获取websocket的h264视频流低延时播放



## 安装ffmpeg 转码：
ffmpeg  -rtsp_transport tcp  -i  "rtsp://admin:guide123@192.168.1.190:554/" -q 0 -buffer_size 1024000  -max_delay 500  -f rawvideo -codec:v  libx264   -preset ultrafast    http://127.0.0.1:8081/supersecret/live3 

## java搭建http服务读取数据流 转websocket通道
参考https://github.com/heshan3662/jsfmpeg/tree/master/nettyDemos/java/http_websocket

## wfs.js
参考https://github.com/ChihChengYang/wfs.js
可编译后得到wfs.js ,也可直接下载本项目wfs.js,有少量代码改动，可比对后调整源码

播放速度  

function h264Demuxer(wfs){...}

function MP4Remuxer(observer, id, config){...}

分别修改上面两函数中的下面两参数的值，其数值越大播放速度会偏慢，经过实验3000比较接近正常。

_this.H264_TIMEBASE = 3000;

this.H264_TIMEBASE = 3000;

 调到视频播放无加载圆圈 ，视频与vlc播放rtsp时间跨度一致即可

 清除缓存   
 
 pushVideo(
…………
  if (this.ISGenerated) {
      // if (videoTrack.samples.length) { 
        this.remuxVideo_2(videoTrack,timeOffset,contiguous);   
      // }
    }  
)

  onH264DataParsed(event){ 
    this._parseAVCTrack( event.data); 
    // if (this.browserType === 1 || this._avcTrack.samples.length >= 20){ // Firefox
      this.remuxer.pushVideo(0, this.sn, this._avcTrack, this.timeOffset, this.contiguous);
      this.sn += 1;
    // }
  } 
  
  修改接口地址
  
   key: 'onMediaAttached',
    value: function onMediaAttached(data) {
      if (data.websocketName != undefined) {
        var client = new WebSocket('ws://127.0.0.1:8081/ws');
        this.wfs.attachWebsocket(client, 'live3');
      } else {
        console.log('websocketName ERROE!!!');
      }
    }
  
  ## 总结
  
  1. 经测试播放画面与vlc 直接播放rtsp视频流稍有延时

  2. 踩过好几天的坑: 
  
  视频画面闪动，是播放速度的原因和ffmpeg指定了转码分辨率导致!
  
  画面有延时wfs缓存，未研究怎么去修改修改大小！测试24帧加载到MP4box进行渲染，修改为1帧。 修改后2s延时
  ffmpeg转码添加 -preset ultrafast  。修改后延时1s内
  
  
  
