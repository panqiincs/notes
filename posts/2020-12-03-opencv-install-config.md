---
title: OpenCV安装和配置
date: 2020-12-03 13:56:29
tags:
  - opencv
  - 树莓派
  - vs2019
  - cmake
categories:
  - 工具使用
---

## 安装

### Windows

按照默认目录安装OpenCV即可。

### 树莓派

请参考文章：[子豪兄教你在树莓派上安装OpenCV](https://zhuanlan.zhihu.com/p/46032511)。

## 配置

### 在VS2019中的配置

添加`C:\opencv\build\x64\vc15\bin`目录到Path环境变量中。

新建一个Visual Studio 2019工程，设置为Debug和x64，添加如下目录：

1. 工程属性--VC++目录--Include目录，加入：`C:\opencv\build\include`
2. 工程属性--VC++目录--库目录，加入：`C:\opencv\build\x64\vc15\lib`
3. 工程属性--链接器--输入--附加依赖项，加入：`opencv_world348d.lib`

### CMake

创建一个测试代码文件，内容如下：

```cpp
#include <stdio.h>
#include <opencv2/opencv.hpp>

using namespace cv;

int main(int argc, char** argv)
{
    if (argc != 2) {
        printf("usage: DisplayImage.out <Image_Path>\n");
        return -1;
    }
    Mat image;
    image = imread(argv[1], 1);
    if (!image.data) {
        printf("No image data \n");
        return -1;
    }
    namedWindow("Display Image", WINDOW_AUTOSIZE);
    imshow("Display Image", image);
    waitKey(0);
    return 0;
}
```

创建CMakeLists.txt文件，内容如下：

```cmake
cmake_minimum_required(VERSION 2.8)
project(DisplayImage)
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})
add_executable(DisplayImage DisplayImage.cpp)
target_link_libraries(DisplayImage ${OpenCV_LIBS})
```
