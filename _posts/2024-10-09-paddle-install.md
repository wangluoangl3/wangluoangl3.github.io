---
layout: post
title: "PaddlePaddle安装"
date: 2024-10-09
catalog: true
tags:
    - PaddlePaddle
---
# 1. 环境准备
## 1.1 创建虚拟环境
### 1.1.1 安装环境
首先根据具体的Python版本创建Anaconda虚拟环境，PaddlePaddle的Anaconda安装支持3.8-3.12版本的Python安装环境。

```python
conda create -n paddle_lw python=3.8
```

### 1.1.2 进入Anaconda虚拟环境

```python
conda activate paddle_lw
```

## 1.2 其他环境检查
### 1.2.1 确认Python安装路径

确认您的conda虚拟环境和需要安装PaddlePaddle的Python是您预期的位置，因为您计算机可能有多个Python。进入Anaconda的命令行终端，输入以下指令确认Python位置。

输出Python路径的命令为：
```bash
which python
```
根据您的环境，您可能需要将说明中所有命令行中的python3替换为具体的Python路径

### 1.2.2 检查Python版本
使用以下命令确认版本

```python
python3 --version
```

### 1.2.3 检查系统环境
确认 Python 和 pip 是 64bit，并且处理器架构是 x86_64（或称作 x64、Intel 64、AMD64）架构。下面的第一行输出的是”64bit”，第二行输出的是”x86_64（或 x64、AMD64）”即可：
```python
python3 -c "import platform;print(platform.architecture());print(platform.machine())"
```

# 2. 开始安装
本文档为您介绍conda安装方式

添加清华源(可选)
对于国内用户无法连接到Anaconda官方源的可以按照以下命令添加清华源：
```python
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
```

```python
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```

```python
conda config --set show_channel_urls yes
```

根据版本进行安装

选择下面您要安装的PaddlePaddle

**CPU 版的 PaddlePaddle**

如果您的计算机没有 NVIDIA® GPU，请安装 CPU 版的 PaddlePaddle
```python
conda install paddlepaddle==3.0.0b1 -c paddle
```
**GPU 版的 PaddlePaddle**

- 对于 CUDA 11.8 安装命令为:

```python
conda install paddlepaddle-gpu==3.0.0b1 paddlepaddle-cuda=11.8 -c paddle -c nvidia
```
- 对于 CUDA 12.3 安装命令为:
```python
conda install paddlepaddle-gpu==3.0.0b1 paddlepaddle-cuda=12.3 -c paddle -c nvidia
```
# 3.验证安装
安装完成后您可以使用 python3 进入 python 解释器，输入import paddle ，再输入 paddle.utils.run_check()

如果出现PaddlePaddle is installed successfully!，说明您已成功安装。
