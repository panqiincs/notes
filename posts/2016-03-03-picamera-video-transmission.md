---
title: 网络传输picamera视频
date: 2016-03-03 22:19:33
tags: 
  - picamera
  - opencv
  - 图像压缩
categories:
  - 程序设计
---

## 需求

在[上一篇文章](https://panqiincs.me/2016/02/18/process-picamera-video-on-pc/)中，我介绍了将树莓派打造成一个rtsp视频流服务器的方法，这样就可以在远程看到树莓派摄像头的实时图像。使用网络上现成的程序固然方便，但丧失了可控性。比如，如果想在远程PC上得到每一帧图像的时间戳，上一篇文章中的方法就无能为力了。本文介绍如何一帧帧读取picamera的图像，压缩后通过网络发送出去的方法。

<!--more-->

## 分析

我使用[RaspiCam](https://github.com/cedricve/raspicam)来获取picamera的图像。这是一个C++库，我使用它的OpenCV接口，将图像数据存储到一个`cv::Mat`对象中。假设采集640x480的彩色图像，帧率为25fps，那么每秒需要传输的数据大小是：640×480×3×25 = 23040000字节，大约22MB，这显然太大了，实测效果也是非常卡。因此图像必须经过压缩再传输。压缩和解压分别使用OpenCV的`imencode`和`imdecode`函数。由于每张图像压缩后大小不一样，所以传输每一帧的图像（压缩过的）之前先发送数据的大小。

## 服务器

下面是服务器端的源代码：

```c
#include <iostream>
#include <vector>
#include <string>
#include <sstream>

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#include <time.h>

#include <opencv2/core/core.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/highgui/highgui.hpp>

#include <raspicam/raspicam_cv.h>

#define  oops(msg)  {perror(msg); exit(1);}

using namespace std;
using namespace cv;

int main(int argc, char **argv)
{
    struct sockaddr_in saddr;
    struct hostent     *hp;
    int    portnum;
    int    listenfd, connfd;

    if (argc < 3) {
        fprintf(stderr, "Usage: %s hostname portnum\n", argv[0]);
        exit(1);
    }
    hp = gethostbyname(argv[1]);
    if (hp == NULL) {
        fprintf(stderr, "No such host!\n");
        exit(1);
    }
    portnum = atoi(argv[2]);

    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
        oops("socket");

    bzero((void *)&saddr, sizeof(saddr));
    bcopy((void *)hp->h_addr, (void *)&saddr.sin_addr, hp->h_length);
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(portnum);

    if (bind(listenfd, (struct sockaddr *)&saddr, sizeof(saddr)) != 0)
        oops("bind");

    if (listen(listenfd, 1) != 0)
        oops("listen");

    // 打开picamera，彩色图像，640x480
    raspicam::RaspiCam_Cv camera;
    camera.set(CV_CAP_PROP_FORMAT, CV_8UC3);
    camera.set(CV_CAP_PROP_FRAME_WIDTH, 640);
    camera.set(CV_CAP_PROP_FRAME_HEIGHT, 480);
    cout << "Openning camera..."; 
    if (!camera.open()) {
        cerr << "Error openning the camera!" << endl;
        return -1;
    }
    cout << "Successfully!" << endl;

    // 压缩参数，jpeg
    vector<int> param = vector<int>(2);
    param[0] = CV_IMWRITE_JPEG_QUALITY;
    param[1] = 95; // default(95) 0-100

    Mat frame; // 存储每一帧图像
    vector<uchar> buff; // 存储压缩后的数据
    char s[20] = {0};

    while (1) {
        connfd = accept(listenfd, NULL, NULL);
        printf("Wow! got a call!\n");
        if (connfd == -1)
            oops("accept");

        for (;;) {
            // 获取图像 
            camera.grab();
            camera.retrieve(frame);
            // 压缩图像
            imencode(".jpg", frame, buff, param);
            int bufflen = buff.size();
            // 发送数据的大小, 将二进制数据转换成文本再发送
            sprintf(s, "%d", bufflen);
            send(connfd, s, 15, 0);
            // 发送
            send(connfd, &buff[0], buff.size(), 0);  
        }
        close(connfd);
    }
}
```

## 客户端

下面是客户端的代码：

```c
#include <iostream>
#include <vector>

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

#define  oops(msg)  {perror(msg); exit(1);}

using namespace std;
using namespace cv;

int main(int argc, char *argv[])
{
    struct sockaddr_in  saddr; 
    struct hostent *hp;
    int    sock; 

    if (argc < 3) {
        fprintf(stderr, "ERROR, no port provided");
        exit(1);
    }
    hp = gethostbyname(argv[1]);
    if (hp == NULL) 
        oops("no such computer");

    sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock == -1) 
        oops("socket");

    bzero(&saddr, sizeof(saddr));
    bcopy(hp->h_addr, (struct sockaddr *)&saddr.sin_addr, hp->h_length);
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(atoi(argv[2]));

    if (connect(sock, (struct sockaddr *)&saddr, sizeof(saddr)) != 0)  
        oops("connect");

    namedWindow("Display image", CV_WINDOW_AUTOSIZE);

    int framelen;
    char s[20] = {0};

    for (;;) {
        // 读取数据的大小，文本转化成int
        recv(sock, s, 15, 0);
        framelen = atoi(s);
        // 接收压缩后的图像数据
        vector<uchar> buff(framelen);
        int bytes = 0;
        for (int i = 0; i < framelen ; i += bytes)
            if ((bytes = recv(sock, &buff[0]+i, framelen-i, 0)) == -1)
                oops("Receive");
        // 解压图像
        Mat frame = imdecode(Mat(buff), CV_LOAD_IMAGE_COLOR);
        imshow("Display image", frame);
        waitKey(20);
    }

    close(sock);    
}
```
