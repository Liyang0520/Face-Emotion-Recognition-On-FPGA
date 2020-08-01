# 基于Ultra96的人脸情感识别系统

## 简介

### 系统概述
人脸情感识别在实践中的应用前景广阔，市场价值也越加凸现。人脸情感识别可以被应用于自动驾驶，公安侦破，电子游戏，教育等许多产业。具体地说，在自动驾驶中可以帮助提升行车安全，通过情感识别可以掌握驾驶者的身体状态，可提供相应的安全防范。

我们实现的基于Ultra96的人脸情感识别系统是一个实时的系统，可以有效进行边缘端的部署（系统框图1示）。首先，我们进行模型设计及训练，利用Keras设计了一个带有可分离卷积的模型进行人脸情感识别，其准确率在验证集上接近90%；然后，我们利用Vitis AI平台，对训得到的权重进行转化、量化和编译得到ELF文件。最后，我们在Ultra96-V2开发板上配置DPU-PYNQ环境，将ELF文件编译到shared library。系统识别的整体流程是OpenCV进行人脸检测，然后对检测到的人脸区域进行情感识别。

### 系统框图
![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/work_framework.png)


## 系统设计
### 设计流程及硬件框架

设计流程

![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/design_framework.png)

硬件框架

![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/HW_framework.png)

### 数据集
本项目开发采用[RaFD](http://www.socsci.ru.nl:8180/RaFD2/RaFD)人脸情感识别，数据集有7个不同的情感分类（生气、开心、快乐、害怕、惊讶、蔑视、中立）（使用学校/学术机构账号申请下载数据集）。

### 网络设计及训练
由于边缘处理器资源的限制，我们设计一个轻量化的网络，由3个标准卷积层、2个可分离卷积层、5个最大池化层、和2个全连接层组成，并且使用Dropout来改善过拟合的问题。

![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/network_framework.png)

运行Ubuntu 16.04系统。软件环境主要为Python 3.6.8, Tensorflow-gpu 1.15.0, Keras 2.2.4。我们训练了200个epoch，并保存最优的模型。训练集和验证集的精确度如图X所示，并最终在验证集上达到了接近90%的准确率。

训练网络命令
```
python ./network/train_fer_mobile.py
```

### 网络量化及编译

我们需要先安装[VitisAI](https://github.com/Xilinx/Vitis-AI)平台。安装完成后，先对.h5权重转化为.pd权重文件。然后，我们使用Vitis AI对之前得到的网络权重进行量化，得到了原先一半大小的网络。最后，我们将其编译为ELF文件。

![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/Vai_Q.png)


进入"vitis-AI"目录
```
cd vitis-AI
```

运行权重转化脚本
```
./1_vitisAI_keras2frozon.sh
```
运行量化脚本
```
./2_vitisAI_tf_quantize.sh
```
运行编译脚本
```
./3_vitisAI_tf_compile.sh
```

### 网络部署
首先我们需要将开发板环境升级到[DPU-PYNQ2.5](https://github.com/Xilinx/DPU-PYNQ)。DPU是一个专门用于卷积神经网络的可编程引擎，PYNQ是基于Python的帮助开发者加快开发的开源框架。DPU-PYNQ是一个结合PYNQ框架和Vitis AI平台开发的全新的系统覆盖层，帮助开发者在Python环境下调取DPU完成深度学习模型的部署。

![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/PYNQ.png)

然后，将Vitis-AI生成的.elf文件再次通过ARM提供的GCC编译成DPU shared library的.so文件。首先进入"deployment"目录，运行转化脚本
```
./1.compile.sh
```

打开'dpuFer.ipynb',我们可以运行得到结果。

## 结果
### Ultra96验证集结果
![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/valid_result.png)

### Ultra96直接识别结果
![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/result.png)

### Ultra96检测+识别结果
![](https://github.com/Liyang0520/Face-Emotion-Recognition-On-FPGA/blob/master/images/result_update.png)


## 参考

[RpsU96: Rock Paper Scissors on Ultra96](https://github.com/xupsh/rps_u96)





