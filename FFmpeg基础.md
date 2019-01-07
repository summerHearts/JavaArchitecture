##播放器架构

![](https://upload-images.jianshu.io/upload_images/325120-2ab1f4fa5d39a89d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  同时还有音视频同步，这个是很重要的。

- 渲染流程

![](https://upload-images.jianshu.io/upload_images/325120-c01d033430967991.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##1、FFmpeg常用命令实战

FFmpeg音视频处理流程讲解

![](https://upload-images.jianshu.io/upload_images/325120-6c96e23a386080e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


![](https://upload-images.jianshu.io/upload_images/325120-ce8791de733964a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- FFmpeg基本信息查询命令

  ![](https://upload-images.jianshu.io/upload_images/325120-5434cd9a675b08d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
  ![](https://upload-images.jianshu.io/upload_images/325120-41d335dbe435fb4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  ![](https://upload-images.jianshu.io/upload_images/325120-e8d8a1722dc00abf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
- FFmpeg录制命令实战

 ```
 ffmpeg -f avfoundation -i 1 -r 30 out.yuv
 ```
 
 - f: 指定使用`avfoundation` 采集数据
 - -i: 指定从哪里采集数据，它是一个文件索引号
 - -r: 指定帧率

 ```
 ffplay -s 2880x1800 out.yuv 
 ```
 
 ![](https://upload-images.jianshu.io/upload_images/325120-fbddd5455af81bd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
 可以看出录制时使用的是  `yuv422p`,播放时使用的是 `yuv420p`像素不一样，无法正确渲染出来。
 
  ```
 ffplay -s 2880x1800  -pix_fmt uyvy422  out.yuv 
 ```
 这样就可以完整的显示出来了。
 
 ```
 ffmpeg -f avfoundation -list_devices  true -i ""
 ```
 
 ![](https://upload-images.jianshu.io/upload_images/325120-5dd87b28ed877aac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
 ```
 ffmpeg -f avfoundation -i :0  out.wav
 ```
 
 :0  代表查询出来的设备编号
 
 ```
 ffplay out.wav
 ```
 
 ![Xnip2018-11-12_10-46-53.png](https://upload-images.jianshu.io/upload_images/325120-12979bd5f83d1b27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
 ![](https://upload-images.jianshu.io/upload_images/325120-c33b69523772381f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


- ffmpeg分解与复用命令
  
  ![](https://upload-images.jianshu.io/upload_images/325120-98438f242db1de78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
  ```
  ffmpeg -i out.mp4 -vcodec copy -acodec copy out.flv 
  ```
  
  - -i :输入文件
  - -vcodec copy: 视频编码处理方式
  - -acodec copy: 音频编码处理方式

      ![](https://upload-images.jianshu.io/upload_images/325120-4b33bbd83cdad215.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
      
  - 抽取出视频
  
      ```
    ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4  -an -vcodec copy out.h264
      ```
  
       ![](https://upload-images.jianshu.io/upload_images/325120-cf5ab17f3ca272c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
  -  抽取出音频

      ```
       ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4  -acodec copy -vn out.aac
      ```
     
- **ffmpeg 处理原始数据命令实战**

  ```
  ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -an -c:v rawvideo -pix_fmt yuv420p outs.yuv
  ```
  
  抽取视频中的yuv数据
  
   ```
  ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -vn -ar 44100 -ac 2 -f s16le out.pcm
   ```
  
   - ar: 视频采样率
   - ac2: 双声道
   - f:数据存储格式

     ![](https://upload-images.jianshu.io/upload_images/325120-d071f378edce988c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
     
     ```
     ffplay -ar 44100 -ac 2 -f s16le out.pcm
     ```

- ffmpeg滤镜命令

   ![](https://upload-images.jianshu.io/upload_images/325120-b72eb44abb61b469.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
   
   ```
   ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -vf crop=in_w+400:in_h+400 -c:v libx264 -c:a copy out4.mp4
   ```
   
   ```
   ffplay out4.mp4
   ```
  
- ffmpeg音视频的裁剪与合并命令
   
   ```
   ffmpeg -i  /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -ss 00:00:50 -t 120 out.ts
   ```
   ```
   file 'ut1.ts'
   file 'ut2.ts'
   ```
   
   ```
   ffmpeg -f concat -i input.txt outA.mp4
   ```

- ffmpeg图片与视频互转

    ```
    ffmpeg -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -r 1 -f image2 image-%3d.png
    ```
    
    ```
    ffmpeg -i image-%3d.png out.mp4
    ```
    
    视频一下就播放完毕

- ffmpeg直播相关的命令

    ```
    ffmpeg -re -i /Users/kevin/Desktop/2018.05.20\ 房东的猫\ 厦门\ 《我可以》.mp4 -c copy -f flv rtmp://server/live/streamName
    ```
    
    ```
    ffmpeg -re -i rtmp://server/live/streamName -c copy dump.flv
    ```

   推流地址： http://www.hangge.com/blog/cache/detail_1325.html

   
   
   
  
  

  
  
  
  





