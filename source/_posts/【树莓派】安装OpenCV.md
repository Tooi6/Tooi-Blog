---
title: 【树莓派】安装OpenCV  
date: 2019-11-18 10:12:40  
tags:  
- OpenCV
---

### 配置摄像头  
- **开启树莓派摄像头设置**
> 查看博客：https://tooi6.github.io/2019/11/15/%E3%80%90%E6%A0%91%E8%8E%93%E6%B4%BE%E3%80%91%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE%20%20/  
 
- **设置设备**  

```
# 添加设备 
sudo nano /etc/modules
# 末尾添加  
bcm2835-v4l2
# 检测
vcgencmd get_camera
# 输出：supported=1 detected=0 连接成功  

# 拍照
raspistill -t 1000 -o image.jpg
```
  
### 开始安装OpenCV

- **准备**  
```
# 安装numpy
sudo pip3 install numpy

# 在树莓派设置中把根目录扩大到整个SD卡
sudo raspi-config
# 选择 Advanced Options  
# Expand Filesystem 
# 重启 reboot  

# 安装所需的库
sudo apt-get install build-essential git cmake pkg-config -y
sudo apt-get install libjpeg8-dev -y
sudo apt-get install libtiff5-dev -y
sudo apt-get install libjasper-dev -y
sudo apt-get install libpng12-dev -y

sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev -y

sudo apt-get install libgtk2.0-dev -y
sudo apt-get install libatlas-base-dev gfortran -y

# 下载OpenCV  
cd /home/pi/Download  
wget https://github.com/Itseez/opencv/archive/3.4.0.zip
wget https://github.com/Itseez/opencv_contrib/archive/3.4.0.zip
# 百度云
# 链接：https://pan.baidu.com/s/1AiiO9bVpkrUl7qUift2H5Q 
# 提取码：san9 

# 解压
unzip opencv-3.4.0.zip
unzip opencv_contrib-3.4.0.zip

# 编译
cd /home/pi/Downloads/opencv-3.4.0
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D OPENCV_EXTRA_MODULES_PATH=/home/pi/Downloads/opencv_contrib-3.4.0/modules -D BUILD_EXAMPLES=ON -D WITH_LIBV4L=ON PYTHON3_EXECUTABLE=/usr/bin/python3.5 PYTHON_INCLUDE_DIR=/usr/include/python3.5 PYTHON_LIBRARY=/usr/lib/arm-linux-gnueabihf/libpython3.5m.so PYTHON3_NUMPY_INCLUDE_DIRS=/home/pi/.local/lib/python3.5/site-packages/numpy/core/include ..
# 等待15分钟
```
根据下图判断是否编译成功    

![image](https://note.youdao.com/yws/api/personal/file/25A5D389ADF749B8B7DA21F79283CFDB?method=download&shareKey=aef794f28d36fa21b07cc6af743ec8c1)

- **开始编译**  

```
# 编译
cd /home/pi/Downloads/opencv-3.4.0/build
make
# 等5个小时。。。

# 成功后安装  
sudo make insall
```

- **测试**

```
python
###
>>> import cv2
>>> cv2.__version__
'3.4.0'
###
```


