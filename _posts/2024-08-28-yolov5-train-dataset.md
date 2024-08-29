---
layout: post
title: yolov5训练数据集-实践操作
data: 2024-08-28
catalog: true
tags:
  - object detection
---

使用yolov5训练自己的数据集，以SDGS数据集为例，参考[^1],[^2],[^3],[^4],[^5]步骤如下：

[**yolov5源码下载**](https://github.com/ultralytics/yolov5)

# 1.环境配置
使用Anaconda创建python环境，并安装yolov5所需的依赖包。

（1）创建虚拟环境并激活，要求python最低版本为3.7

```python
conda create -n yolov5 python==3.8
conda activate yolov5
```

（2）安装依赖包

先切换到项目路径
>切换盘： 盘名
进入盘内路径： cd + 路径

执行安装：
```python
pip install -r requirements.txt
```

# 2.数据集准备
yolov5需要的数据集格式为voc格式，将图片和标签文件保存于images、annotations两个文件夹中，其中annoations中存放xml格式的标签文件。
>path/SDGS
— annotations
— images
## 2.1 划分训练/验证集
运行下列代码，划分训练集、验证集。
修改xmlFilePath,saveBasePath为自己的数据集路径，第二个路径若不存在会自己创建。
```python
import os
import random

random.seed(10)  # 设置随机数种子,复现随机场景所必须的

xmlFilePath = 'pathto/SDGS/annotations'
saveBasePath = 'pathto/SDGS/ImageSets/Main/'

trainval_percent = 1  # trainval_percent=0.9# 表示余下的百分之十用于test,为1则不划分测试集
train_percent = 0.8  # train_percent=1 # 表示训练集中用于训练，没有用于验证

temp_xml = os.listdir(xmlFilePath)  # 获得一个列表,每个元素是一个文件名
total_xml = []  # 用于保存所有xml文件的文件名
for xml in temp_xml:  # 遍历文件夹下所有文件
    if xml.endswith(".xml"):  # 判断文件名是否以.xml结尾
    #if xml.endswith(".txt"):  # 判断文件名是否以.txt结尾
        total_xml.append(xml)
if not os.path.exists(saveBasePath):
    os.makedirs(saveBasePath)
    
num = len(total_xml)  # 所有xml文件的总数
indices = list(range(num))  # 获得迭代类型,0 ~ (num-1)
tv = int(num * trainval_percent)  # 用于训练和验证的数量
tr = int(tv * train_percent)  # 用于训练的数量
trainval = random.sample(indices, tv)  # 用于训练和验证的样本的索引
train = random.sample(trainval, tr)  # 用于训练的样本的索引

print("train and validation set size:", tv)  # 训练样本和验证样本的总数
print("train set size:", tr)  # 训练样本的数量
ftrainval = open(saveBasePath+'trainval.txt','w')  # 依次打开4个文件
ftest = open(saveBasePath+'test.txt', 'w')
ftrain = open(saveBasePath+'train.txt', 'w')
fval = open(saveBasePath+'val.txt', 'w')

for i in indices:
    name = total_xml[i][:-4] + '\n'  # 文件名+'\n',其中文件名不含.xml
    if i in trainval:  # 训练集和验证集的索引
        ftrainval.write(name)  # 写入训练和验证的文件中
        if i in train:  # 训练集的索引
            ftrain.write(name)  # 写入训练的文件中
        else:
            fval.write(name)  # 写入验证的文件中
    else:
        ftest.write(name)  # 否则归于测试集,写入测试的文件中

ftrainval.close()  # 依次关闭4个文件
ftrain.close()
fval.close()
ftest.close()
```

## 2.2 标签格式转换
yolo要求的标签格式为txt，内容为：
`class_id, x_center, y_center, w, h`
将xml转为yolo所需的txt文件，代码参考[Yolov5训练自己的数据集](https://blog.csdn.net/qq_45945548/article/details/121701492)

```python
# -*- coding: utf-8 -*-
import xml.etree.ElementTree as ET
import os
from os import getcwd

sets = ['train', 'val', 'test']
classes = ["aircraft", "oiltank","overpass",'playground']  # 改成自己的类别
abs_path = os.getcwd()
print(abs_path)


def convert(size, box):
    dw = 1. / (size[0])
    dh = 1. / (size[1])
    x = (box[0] + box[1]) / 2.0 - 1
    y = (box[2] + box[3]) / 2.0 - 1
    w = box[1] - box[0]
    h = box[3] - box[2]
    x = x * dw
    w = w * dw
    y = y * dh
    h = h * dh
    return x, y, w, h


def convert_annotation(image_id):
    in_file = open('pathto/SDGS/Annotations/%s.xml' % (image_id), encoding='UTF-8')
    out_file = open('pathto/SDGS/labels/%s.txt' % (image_id), 'w')
    tree = ET.parse(in_file)
    root = tree.getroot()
    size = root.find('size')
    w = int(size.find('width').text)
    h = int(size.find('height').text)
    for obj in root.iter('object'):
        difficult = obj.find('difficult').text
        # difficult = obj.find('Difficult').text
        cls = obj.find('name').text
        if cls not in classes or int(difficult) == 1:
            continue
        cls_id = classes.index(cls)
        xmlbox = obj.find('bndbox')
        b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text), float(xmlbox.find('ymin').text),
             float(xmlbox.find('ymax').text))
        b1, b2, b3, b4 = b
        # 标注越界修正
        if b2 > w:
            b2 = w
        if b4 > h:
            b4 = h
        b = (b1, b2, b3, b4)
        bb = convert((w, h), b)
        out_file.write(str(cls_id) + " " + " ".join([str(a) for a in bb]) + '\n')


wd = getcwd()
for image_set in sets:
    if not os.path.exists('pathto/SDGS/labels/'):
        os.makedirs('pathto/SDGS/labels/')
    image_ids = open('pathto/SDGS/ImageSets/Main/%s.txt' % (image_set)).read().strip().split()

    if not os.path.exists('pathto/SDGS/dataSet_path/'):
        os.makedirs('pathto/SDGS/dataSet_path/')

    list_file = open('pathto/SDGS/dataSet_path/%s.txt' % (image_set), 'w')
    # 这行路径不需更改，这是相对路径
    for image_id in image_ids:
        list_file.write('pathto/SDGS/images/%s.jpg\n' % (image_id))
        convert_annotation(image_id)
    list_file.close()

```
## 2.3 创建配置文件
在项目data文件夹下新建自己的配置文件SDGS.yaml,内容如下，路径为自己的数据集路径，设置类别数，类别名称。注意此处的<font style="微软雅黑" color=pink>names和标签格式转换部分的代码中classes要保持一致</font>.
```python
train: pathto/SDGS/dataSet_path/train.txt
val: pathto/SDGS/dataSet_path/val.txt

# number of classes
nc: 4

# class names
names: ["aircraft", "oiltank","overpass",'playground']
```
# 3.训练
（1）修改配置

直接在train.py的函数parse_opt中修改配置，修改如下：
修改epochs，batchsize，根据自己的配置来，若报内存溢出就把batch-size调小一点，一般为2的倍数，修改weights为预训练文件路径，修改data为自己的数据配置yaml文件路径，为了防止出错建议全部使用<font style="微软雅黑" color=red>绝对路径</font>。
# 4.验证
训练到最后模型会输出在验证集上的评估结果，包括数据集参数、精度参数和模型参数，测试时会另外给出推理速度相关参数。
>数据集参数：'Class', 'Images', 'Instances'
精度参数：'P', 'R', 'mAP50', 'mAP50-95'
模型参数： layers, parameters, gradients, GFLOPS
速度参数：ms pre-process, ms inference, ms NMS per image at shape

修改val.py, parse_opt()中的data, weights, batch-size或使用命令行
`python val.py --data SDGS.yaml --weights runs/train/weights/best.pt`
训练和验证时得到的模型参数结算结果是不同的，参考链接4[^4]
# 5.测试
修改detect.py, parse_opt中，修改weights为训练得到的权重路径，修改source为要用来预测的图片文件路径，data为数据集yaml文件路径，预测得到的图片保存路径为project，也可以修改run.


[^1]:[ImportError: cannot import name ‘OrderedDict‘ from ‘typing‘](https://blog.csdn.net/weixin_46202290/article/details/127886019)
[^2]:[Yolov5训练自己的数据集（详细完整版）](https://blog.csdn.net/qq_45945548/article/details/121701492)
[^3]:[史上最详细yolov5环境配置搭建+配置所需文件](https://blog.csdn.net/qq_44697805/article/details/107702939)
[^4]:[YOLOv5-6.x 模型参数量param及计算量FLOPs解析](https://blog.csdn.net/weixin_43799388/article/details/123914837)
[^5]:[【yolov5】训练自己的数据集-实践笔记](https://blog.csdn.net/gsgs1234/article/details/131551955)
