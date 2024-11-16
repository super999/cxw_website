---
title: Jetson TX2 NX 部署安装过程
date: 2024-10-07 11:19:49
tags:
  - Jetson TX2 NX
  - 部署
  - 安装
permalink: /posts/jetson-tx2-nx-deployment-installation-process.html
top_img: /cnblog/img/post/101/cover.jpg
cover: /img/post/101/cover.jpg
categories: 
  - Jetson TX2 NX
keywords: Jetson TX2 NX, 部署, 安装
---

## 先安装给单片机操作系统

### 安装虚拟机

[参考链接 https://www.yahboom.com/build.html?id=5358&cid=530](https://www.yahboom.com/build.html?id=5358&cid=530)

### 安装 VMWare Workstation

![VMWare Workstation](/img/post/101/虚拟机安装.png)

### 安装 Ubuntu 18.04

![Ubuntu 18.04](/img/post/101/安装ubuntu18.04.png)

### 安装 Nvidia SDK Manager

![Nvidia SDK Manager](/img/post/101/安装nvidia-sdk-manager.png)

1. 打开NVIDIA的jetpack下载网址：
   https://developer.nvidia.com/zh-cn/embedded/jetpack


2. 安装SDK Manager。

先进入刚刚下载的.deb文件的路径，例如这里下载到Downloads目录。

cd Downloads/

![img.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20220408/1649413898655670.png)

```bash
sudo dpkg -i sdkmanager_2.1.0-11698_amd64.deb 
sudo apt --fix-broken install
```

![img.png](/img/post/101/shell_结果.png)

3. 打开Ubuntu18.04系统的程序，搜索SDK，可以找到SDKManager，打开文件。

![Nvidia SDK Manager](/img/post/101/安装nvidia-sdk-manager-2.png)

登录NVIDIA账号，会在浏览器弹出链接，输入用户名和密码登录进去。

![image.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20220408/1649413912364379.png)

4.虚拟机Ubuntu18.04连接Jetson TX2 NX

此时需要让Jetson TX2 NX进入系统REC刷机模式。

将跳线帽连接到FC REC和GND引脚，也就是连接到核心板下方载板的第二和第三个引脚，如下图所示：

![image.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20220408/1649413916148315.png)
![image.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20220408/1649413918406158.png)

连接线路，将HDMI显示屏、鼠标、键盘和microUSB数据线连接到Jetson TX2 NX上，最后再接入电源。由于上一步已经将跳线帽连接FC
REC和GND引脚，所以上电开机后会自动进入REC刷机模式。

5.在虚拟机Ubuntu18.04的SDKManager软件选择Target Hardware为Jetson TX2 NX，JetPack版本，这里以4.6版本为例。
![image.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20220408/1649413926377502.png)

### 把系统移动到固态硬盘

1. 安装rootOnNVME软件

打开NX的终端，在用户目录下输入以下代码

```bash
git clone https://github.com/jetsonhacks/rootOnNVMe.git

# 进入rootOnNVME目录并查看文件

cd rootOnNVMe/
ls
```

![img.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20210811/1628662881432647.png)

2. 复制系统文件

输入以下命令复制文件到M.2固态硬盘。

```bash
./copy-rootfs-ssd.sh
```

![img.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20210811/1628662885253875.png)

3.启动服务，运行后输入NX的密码，按回车键确认，看到以下信息就表示系统成功移动到M.2固态硬盘。

```bash
./setup-service.sh
```

![img.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20210811/1628662887946105.png)

5. 打开TX2 NX的终端，输入以下命令查看储存空间。

```bash
df -h
```

![img.png](https://www.yahboom.com/Public/ueditor/php/upload/image/20210811/1628662895992492.png)

## 修改交换空间的大小

```bash
sudo fallocate -l 4G /var/swapfile
sudo chmod 600 /var/swapfile
sudo mkswap /var/swapfile
sudo swapon /var/swapfile
```

### 安装 jetson-stats

```bash
sudo pip3 install jetson-stats
```

## 安装 torch

### 迅雷下载 torch-1.10.0-cp36-cp36m-linux_aarch64.whl

下载地址： https://nvidia.app.box.com/public/static/fjtbno0vpo676a25cgvuqc1wty0fkkg6.whl

### 修改文件名, 然后安装

```bash 
mv fjtbno0vpo676a25cgvuqc1wty0fkkg6.whl torch-1.10.0-cp36-cp36m-linux_aarch64.whl
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev libomp-dev
pip3 install 'Cython<3'
pip3 install numpy
pip3 install torch-1.10.0-cp36-cp36m-linux_aarch64.whl

# 安装torchvision
sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev libopenblas-dev libavcodec-dev libavformat-dev libswscale-dev
pip3 install -U pillow
pip3 install torchvision==0.11.1
```

### 验证安装

```python
import torch
print(torch.__version__)
print('CUDA available: ' + str(torch.cuda.is_available()))
print('cuDNN version: ' + str(torch.backends.cudnn.version()))
a = torch.cuda.FloatTensor(2).zero_()
print('Tensor a = ' + str(a))
b = torch.randn(2).cuda()
print('Tensor b = ' + str(b))
c = a + b
print('Tensor c = ' + str(c))
```

```python
import torchvision
print(torchvision.__version__)
```
![img.png](/img/post/101/验证Python_torch_torchvision.png)