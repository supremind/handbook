
在对AVA平台相关基础概念有一定了解后，本文会带领大家在深度学习平台上使用平台提供的公开镜像和示例训练代码，以cifar10作为训练数据集，进行一次完整的训练。

# 创建训练
使用AVA平台，可以训练出满足您需求的模型，训练完成后，可以发布出来进行调用（发布的功能正在紧张开发中）。而训练一个模型，一般需要以下3步。
![](https://odum9helk.qnssl.com/FoEs6CA_liRbj8CfNnkP5cIzCOjz)

* 选择数据集
* 选择镜像
* 选择训练资源

下面讲介绍如何使用AVA平台提供的公开数据集cifar10和Mxnet公开镜像来训练一个模型。

在左侧栏点击“训练”进入到训练模块，点击“新建训练”就可以创建一个训练。
![](https://odum9helk.qnssl.com/FkBimb6lcCo0I282n7hyFeKl4UYN)
输入训练名称“cifar10-demo”和描述（可选）“This is a training demo for cifar10”
![](https://odum9helk.qnssl.com/Fv200mjkaIcY8SNtfzm_49Br_deT)

## 选择数据集
AVA平台提供了很多公开数据集供用户使用，在数据集模块的公开数据集tab就可以找到数据集名称为“cifar10”的数据集。cifar10数据集所包含的内容，请[参考](https://www.cs.toronto.edu/~kriz/cifar.html)。
![](https://odum9helk.qnssl.com/FvzOzoBXVFZl5FC-GmZ53PBrU9g0)

训练模型的框架比较多，目前AVA平台能支持的训练框架包括Caffe和Mxnet。使用这些框架进行模型训练时，使用的数据集一般是需要提前生成为特定的格式，例如，Caffe需要lmdb格式的数据，而Mxnet需要recordio格式的数据，当然也可以使用原图格式。

所以，在开始训练之前，需要把数据集格式化成需要的格式。在此示例中，我们将使用平台提供的Mxnet的公开镜像，所以这里数据集需要格式化为recordio的数据。平台已经把cifar10的这50000条数据按照比例格式化为recordio的训练集（40000条）和验证集（10000）。数据格式化成功完成后，就可以在训练中使用了。
![](https://odum9helk.qnssl.com/Fq1kt6UtwTvME4CT7QxoUgG-cQfR)

在训练模型时，我们一般需要为这个训练选择“训练集”和“验证集”，训练集和验证集可以分别来自于不同的数据集的格式化数据。
在本示例中，我们将选择3.1中的公开数据集“cifar10”格式化的recordio的数据，如下图选中百分比为80%的这条格式化数据作为训练集，
![](https://odum9helk.qnssl.com/FpQMK3lVn6ryLNGjoodFJWcaaKJw)
一般情况，我们会把一个数据集拆分成2部分，一部分作为训练集，一部分作为验证集，所以当您选择cifar10的数据集中那条百分比为80%的格式化数据时，系统会提示您是否要选择同时格式化20%的格式化数据作为验证集。点击“确定”按钮，则选中了训练需要的训练集和验证集。
![](https://odum9helk.qnssl.com/FpWwMrybovu6EQHR6ORo8k-YYc9A)

## 选择镜像
镜像提供用户训练模型的环境，用户可以自己构建镜像，也可以使用公开镜像。AVA平台提供了很多公开镜像供用户使用，在本示例中我们将使用mxnet框架的镜像[官方代码](https://github.com/apache/incubator-mxnet/tree/master/example/image-classification)，您可以在镜像模块的公开镜像tab找到“ava-mxnet-py27-gpu”的镜像,请选择版本为“latest”的镜像。此公开镜像自带AVASDK以及用AVASDK编写的示例训练代码，代码路径为“/workspace/examples/trainings/mxnet/simple/start.sh”，代码中对cifar10数据集，使用ResNet50的网络结构训练模型进行训练。

![](https://odum9helk.qnssl.com/Fm8qpjByEjYYo1odgYVoJ1eQz02j)

点击“下一步”，进入到选择训练资源

## 选择训练资源
在训练资源页面，用户可以选择GPU资源或CPU资源，在这里可以选择GPU资源，目前仅提供1张GPU资源。
执行入口填入的是指定执行的主代码文件，必填，这里填入的是示例代码的路径：/workspace/examples/trainings/mxnet/simple/start.sh
![](https://odum9helk.qnssl.com/FhpRjNFJKdi9WU794bvZe3Gror_h)
点击“创建”按钮，可以看到这个训练就处于“创建中”，意味着分配好资源后。当显示“执行中”，代表系统开始执行训练。
![](https://odum9helk.qnssl.com/FgnvPp0xE3mnS3zCDqEAqLjMCmfa)


训练执行过程中，您可以在平台中查看训练监控，查看训练日志和训练产生的模型。

## 查看训练监控
在训练列表中，找到你刚才创建的训练“cifar10-demo”,然后点击“训练监控”。
![](https://odum9helk.qnssl.com/FrWT7HSHF8_15fZfyi2DJEq_KNSl)
即可查看这个训练的accuracy\loss\learning rate曲线图。
![](https://odum9helk.qnssl.com/FmeAaR6PeHktOa8KqhbOPn9Blx3E)


## 查看训练日志
对于正在执行中的训练，或者已经完成的训练，用户都可以查看训练的日志。现在训练的日志是存储在组共享存储空间的“/workspace/mnt/group/qiniu-group-1/avatest/run/train_cifar10-demo_out.log”文件中，查看训练日志，需要创建一个工作台，通过工作台登录到JupyterLab中，然后通过terminal进入到组共享存储空间去查看日志纪录。工作台的详细功能，请点击[工作台](https://developer.qiniu.io/ava/manual/4479/workspace)。


# 训练产生的模型
对于训练产生的模型，可以在AVA平台的模型模块->训练产生模型tab进行查看。找到上面创建的训练“cifar10-demo”,点击左边的展开加号，就可以看到训练中指定保存的训练产生模型。
您可以下载满足您需求的模型。
![](https://odum9helk.qnssl.com/FgCh5xBtPChY2Swl5lDGF0SMkbsC)

恭喜您，至此，您就在AVA平台上训练出了一个模型。



