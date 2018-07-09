# 训练图像检测模型

SSD，英文全称 Single Shot Multibox Detector，是用于目标检测的算法。常见的目标检测算法，如 Faster R-CNN ，存在着速度慢的缺点。SSD 不仅提高了速度，而且提高了准确度。本教程会通过训练模型、评估模型并且演示一个实例，为 AVA 深度学习平台使用者提供在 MXNET 框架上，使用 SSD 模型对目标进行检测。

### **环境准备**

#### **创建**AVA工作台

1.从工作台标签进入，选择【新建工作台】

  输入名称“SSDtest”，点击确认 

  ![](/images/ch-08/newWorkspace.png)

2.选择数据集
（可以跳过这一步）

3.选择所需要的镜  reg.qiniu.com/ava-public/ava-mxnet:py35-cuda80-cudnn7，然后点击下一步 

  ![](/images/ch-08/py35-cuda80-cudnn7.png)

4.选择计算资源

  此处选择 GPU ，然后选择 4 核 60 G 内存

  ![](/images/ch-07/ava_resource.png)

5.点击创建，等待工作台创建完成

#### 准备

1.如果你使用的是 mxnet-python API，你可能已经有了 python 模块，如果没有的话，此处我们需要安装：cv2, matplotlib 和 numpy，使用 apt-get 命令：

  首先进行 apt-get update 操作： 
  ![](/images/ch-08/aptGetUpdate.png)

  再进行`sudo apt-get install python-opencv python-matplotlib python-numpy` 
  ![](/images/ch-08/installPythonPakage.png)

2.由于我们要使用 MXNET 来运行 SSD 训练，进一步配置：

  ``` bash
  $ git clone [git@github.com](/var/folders/sc/klgj6bt56195l0nyqzhjw5080000gn/T/abnerworks.Typora/52D8EA54-2FF8-4A92-B1C7-B2BFCF391F8D/mailto:git@github.com):apache/incubator-mxnet.git

  $ cd incubator-mxnet

  $ cp -r example /workspace/mnt/group/<your-group>/<your-name>/.
```

  把 example 目录拷贝的自己的共享目录, 方便运行。

#### **训练模型**

1.此示例仅适用于在 Pascal VOC 数据集上的训练。首先下载数据集，再将他们解压至 model 文件夹目录下： 

  ![](/images/ch-08/dataset.png)

  得到： 

  ![](/images/ch-08/unzip.png)

  接着下载 Pascal VOC 数据集：

  ```
  $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar`

  $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar`

  $ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar`
```

  解压：

  ```
  $ tar -xvf VOCtrainval_11-May-2012.tar`

  $ tar -xvf VOCtrainval_06-Nov-2007.tar`

  $ tar -xvf VOCtest_06-Nov-2007.tar`
```

  我们要使用 VOC20017 和 VOC2012 里的 trainval 数据集， VOC2007 和 VOC2012 都在 VOCdevkit 文件夹下，然后将 VOCdevkit 和 data / VOCdevkit 两个文件夹链接到一起， 
  ![](/images/ch-08/link.png)

  创建训练：

  `bash tools/prepare_pascal.sh`

  开始训练： 

  ![](/images/ch-08/train.png)

  可以看到每个 batch 训练的速度和损失： 
  ![](/images/ch-08/trainOutcome.png)

  默认情况下，此示例使用的 batch-size 为 32，learning_rate 为 0.002. 我们可以根据不同的配置来微调一下其中的参数，例如，如果你用的是四核的 GPU ：
  `python3 train.py --gpus 0,1,2,3 --batch-size 32.`

  输入 `python3 demo.py —help` 获得更多帮助

#### **评估训练模型**

确保 val.rec 为验证数据集，运行 `python3 evaluate.py --gpus 0,1 --batch-size 128 --epoch 2`

运行结果如下图。可以看到由于数据数量较少，平均准确率 MAP（Mean Average Precision）较低：

![](/images/ch-08/evaluate.png)


#### **示例**

1.首先下载模型 [ssd_resnet50_0712.zip](https://github.com/zhreshold/mxnet-ssd/releases/download/v0.6/resnet50_ssd_512_voc0712_trainval.zip)，并将其拷贝至你的 model 目录下并解压:  
apt-get install zip
![](/images/ch-08/installMode.png)
![](/images/ch-08/unzip2.png)

解压后，ls查看文件夹内得到了 ssd_resnet50_512-0000.params 和 ssd_resnet50_512-symbol.json 

![](/images/ch-08/unzip2Outcome.png)

2.运行

接着下载测试要用的图片数据集 
![](/images/ch-08/demoDataset.png)

然后运行示例： 
![](/images/ch-08/runDemo.py.jpg)
如果出现 No module named ‘_tkinter’ 的错误，说明没有安装 python3-tk 包，进行安装：`apt-get install python3-tk`

可以看到 demo.py 运行成功后，目录下出现 tmp.png，此处在Python3 notebook 内用代码输出出片结果：

![](/images/ch-08/demoImgShow.png)

继续运行：

`Python3 demo.py --epoch 0 --images ./data/demo/dog.jpg --thresh 0.5`

`Python3 demo.py --cpu --network resnet50 --data-shape 512`

结果： 

![](/images/ch-08/last.png)

你可以输入 `python demo.py —help` 获得更多帮助