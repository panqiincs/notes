---
title: 在PC上显示picamera视频
date: 2016-02-18 11:38:58
tags:
  - picamera
  - opencv
categories:
  - 程序设计
---

## 需求

我想实现用树莓派摄像头picamera采集数据，发送到远程主机上显示或进行其他处理。[picamera示例程序](https://picamera.readthedocs.org/en/release-1.10/recipes1.html#recording-to-a-network-stream)中已经有相应的实现。由于项目需要，远程PC端要使用OpenCV库处理图像，我尝试在树莓派上使用picamera库都没有得到满意的效果。

## 服务器

终于找到了一个用C++写的[rtsp](https://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol)服务器程序——[h264_v4l2_rtspserver](https://github.com/mpromonet/h264_v4l2_rtspserver)。首先运行如下命令：

<!--more-->

```bash
sudo apt-get install v4l-utils
sudo modprobe bcm2835-v4l2
```

这样就可以通过`/dev/video0`设备文件来获取摄像头的数据。注意每次重启后都要运行第二条命令，如果想省略这一步，需要在`/etc/modules`添加一行：`bcm2835-v4l2`。

安装`h264_v4l2_rtspserver`之后运行如下命令：

```bash
sudo h264_v4l2_rtspserver/h264_v4l2_rtspserver -F 15 -W 800 -H 600 -P 8555 /dev/video0
```

即在树莓派上运行了一个rtsp服务器，图像大小是800x600，fps是15，端口号为8555。


## 客户端

在远程PC上打开vlc播放器并输入如下url即可看到视频：

```bash
rtsp://ip-address-of-your-rPI:8555/unicast
```

OpenCV的[VideoCapture](http://docs.opencv.org/2.4/modules/highgui/doc/reading_and_writing_images_and_video.html)类也可以读取上述视频流。下面的源代码实现了打开一个rtsp链接，读取并显示每一帧图像：

```cpp
#include <iostream>
#include <string>
#include <iomanip>
#include <sstream>

#include <opencv2/core/core.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace std;
using namespace cv;

int main(int argc, char *argv[])
{
    const string sourceUrl = "rtsp://ip-address:8555/unicast"; 
    int delay = 20;
    int frameNum = -1;          // Frame counter
    char c;

    VideoCapture capt(sourceUrl);
    if (!capt.isOpened()) {
        cout  << "Could not open url " << sourceUrl << endl;
        return -1;
    }
    Size refS = Size((int) capt.get(CV_CAP_PROP_FRAME_WIDTH),
                     (int) capt.get(CV_CAP_PROP_FRAME_HEIGHT));

    const char* WIN_RF = "RTSP";
    // Windows
    namedWindow(WIN_RF, CV_WINDOW_AUTOSIZE);

    cout << "Frame resolution: Width=" 
         << refS.width << "  Height=" << refS.height << " of nr#: " 
         << capt.get(CV_CAP_PROP_FRAME_COUNT) << endl;

    Mat frame;

    // Show the image captured in the window and repeat
    for (;;) {
        capt >> frame;
        if (frame.empty()) {
            cout << " < < <  Game over!  > > > " << endl;
            break;
        }
        ++frameNum;
        cout << "Frame: " << frameNum << "# " << endl;
        imshow(WIN_RF, frame);
        c = (char)cvWaitKey(delay);
        if (c == 27)
            break;
    }

    return 0;
}
```

## 参考

1. [Making a Raspberry Pi HD Camera](http://hpcc.info/discussion/30/making-a-raspberry-pi-hd-camera)
2. [Video Input with OpenCV and similarity measurement](http://docs.opencv.org/2.4/doc/tutorials/highgui/video-input-psnr-ssim/video-input-psnr-ssim.html#videoinputpsnrmssim)
