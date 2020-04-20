---
title: 采集Hokuyo激光雷达数据
date: 2015-12-05 17:20:24
tags:
  - hokuyo
categories:
  - 工具使用
---

## hokuyoaist

通过阅读Hokuyo数据手册中关于通信协议的内容就可以自己写程序来获取数据了。如果不想麻烦，可以使用别人写好的库——[hokuyoaist](http://gbiggs.github.io/hokuyoaist/)，它支持多个型号，亲测在UTM-30LX型号的激光雷达上可用。

在[Github](https://github.com/gbiggs/hokuyoaist)上下载它的源代码并编译：

<!--more-->

```bash
git clone https://github.com/gbiggs/hokuyoaist.git
cd hokuyaoaist
mkdir build
cd build
cmake ..
make
```

编译会有错误，提示缺少[flexiport](http://gbiggs.github.io/flexiport/)，这个库和`hokuyoaist`出自同一个作者。将它下载下来，编译并安装：

```bash
git clone https://github.com/gbiggs/flexiport.git
cd flexiport
mkdir build
cd build
cmake ..
make && sudo make install
```

`Flexiport`就被安装到指定的系统目录中去了，无需在`CMakeLists.txt`中设置选项就能使用它。

再次重复`hokuyoaist`的编译步骤，又看到了新的错误，提示一个名为`sphinx_doc`的target编译不过，这个是生成文档的语句，无关紧要，可以在`hokuyoaist`的`CMakeList.txt`中将相关的行注释掉，打开`hokuyoaist`根目录下的 `CMakeLists.txt`文件，注释如下三行：

```cmake
if(BUILD_DOCUMENTATION)
    add_subdirectory(doc)
endif(BUILD_DOCUMENTATION)
```

这样就能编译成功了。

## 测试结果

`hokuyoaist`的`example`目录下有采集一次数据的示例程序，编译后，`build`文件夹下会生成一个名为`hokuyoaist_example`的可执行文件。

激光雷达连接到计算机之后，会出现一个名为`/dev/ttyACM0`的设备文件，修改该文件的权限：

```bash
sudo chmod a+rw /dev/ttyACM0
```

运行`hokuyoaist_example`，参数都是默认值：

```bash
./hokuyoaist_example
```

可以看到屏幕上打印的结果，开头是激光雷达的基本信息，接下来是1081个测距值（单位是mm），如果检测到了不合理的值（例如某方向在最大距离内没有检测到障碍物），还会有错误提示。
