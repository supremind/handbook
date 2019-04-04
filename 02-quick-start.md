
在对 AVA 平台相关基础概念有一定了解后, 本文会带领大家在深度学习平台上使用平台提供的公开镜像和示例训练代码, 以 cifar10 作为训练数据集, 进行一次完整的训练, 此次训练的目的是对图像进行分类.

# 创建训练
使用 AVA 平台. 可以训练出满足您需求的模型. 训练完成后, 可以发布出来进行调用(发布的功能正在紧张开发中). 而训练一个模型, 一般需要以下3步.

![](https://odum9helk.qnssl.com/FoEs6CA_liRbj8CfNnkP5cIzCOjz)

下面将介绍如何使用 AVA 平台提供的 **MXNet框架 Cifar10数据集** 来训练一个模型.

在左侧栏点击"训练”进入到训练模块, 点击"新建训练”就可以创建一个训练.

![](https://odum9helk.qnssl.com/FkBimb6lcCo0I282n7hyFeKl4UYN?imageView2/2/w/800/)

输入训练名称"cifar10-demo”和描述(可选)"This is a training demo for cifar10”

![](https://odum9helk.qnssl.com/Fv200mjkaIcY8SNtfzm_49Br_deT?imageView2/2/w/800/)

## 选择数据集
AVA 平台提供了很多公开数据集供用户使用, 在数据集模块的公开数据集 tab 就可以找到数据集名称为"cifar10”的数据集. cifar10 数据集所包含的内容, 请[参考](https://www.cs.toronto.edu/~kriz/cifar.html).

![](https://odum9helk.qnssl.com/FvzOzoBXVFZl5FC-GmZ53PBrU9g0?imageView2/2/w/800/)

训练模型的框架比较多, 目前 AVA 平台能支持的训练框架包括 Caffe\MXNet\Tensorflow 和 Pytorch. 使用这些框架进行模型训练时, 使用的数据集一般是需要提前生成为特定的格式, 例如, Caffe 需要 lmdb 格式的数据, 而 MXNet 需要 recordio 格式的数据, 当然也可以使用原图格式.

所以, 在开始训练之前, 需要把数据集格式化成需要的格式. 在此示例中, 我们将使用平台提供的 MXNet 的公开镜像, 所以这里数据集需要格式化为 recordio 的数据. 平台已经把 cifar10 的这50000条数据按照比例格式化为 recordio 的训练集(40000条)和验证集(10000). 数据格式化成功完成后, 就可以在训练中使用了.

![](https://odum9helk.qnssl.com/Fq1kt6UtwTvME4CT7QxoUgG-cQfR?imageView2/2/w/800/)

在训练模型时, 我们一般需要为这个训练选择"训练集”和"验证集”, 训练集和验证集可以分别来自于不同的数据集的格式化数据.
在本示例中, 我们将选择3.1中的公开数据集"cifar10”格式化的 recordio 的数据, 如下图选中百分比为80%的这条格式化数据作为训练集, 一般情况, 我们会把一个数据集拆分成2部分, 一部分作为训练集, 一部分作为验证集, 所以当您选择 cifar10 的数据集中那条百分比为80%的格式化数据时, 系统会提示您是否要选择同时格式化20%的格式化数据作为验证集. 点击"确定”按钮, 则选中了训练需要的训练集和验证集.

## 选择镜像
镜像提供用户训练模型的环境, 用户可以自己构建镜像, 也可以使用公开镜像. AVA 平台提供了很多公开镜像供用户使用, 在本示例中我们将使用 MXNet 框架的镜像[官方代码](https://github.com/apache/incubator-MXNet/tree/master/example/image-classification), 您可以在镜像模块的公开镜像 tab 找到需要的版本, 此处我们选择 ava-mxnet 中”1.4.0-cuda90”的镜像. (注意：训练选择的镜像需与工作台选择的镜像对应, 否则训练会创建失败.) 此公开镜像自带示例训练代码, 代码路径为"/workspace/examples/trainings/MXNet/simple/start.sh”, 代码中对 cifar10 数据集, 使用ResNet50的网络结构训练模型进行训练.

![](/images/ch-02/public-images.png)

点击"下一步”, 进入到选择训练资源

## 选择训练资源
在训练资源页面, 用户可以选择 GPU 资源或 CPU 资源, 在这里可以选择 GPU 资源.
执行入口填入的是指定执行的主代码文件, 必填, 这里填入的是示例代码的路径：/workspace/examples/trainings/MXNet/simple/start.sh

![](/images/ch-02/settings.png)

点击"创建”按钮, 可以看到这个训练就处于"创建中”, 意味着分配好资源后. 当显示"执行中”, 代表系统开始执行训练.

![](/images/ch-02/job-created.png)

训练执行过程中, 您可以在平台中查看训练监控, 查看训练日志和训练产生的模型.

## 查看训练日志
在训练列表中, 找到你刚才创建的训练, 然后点击红框图标查询训练日志.

![](/images/ch-02/job-logger.png)

对于正在执行中的训练, 或者已经完成的训练, 用户都可以查看训练的日志. 训练的日志是存储在组共享存储空间的`/workspace/mnt/group/<group-name>/<user-name>/run/`文件中, 查看训练日志, 需要创建一个工作台, 通过工作台登录到 JupyterLab 中, 然后通过 terminal 进入到组共享存储空间去查看日志纪录.工作台的详细功能, 请点击[工作台](https://developer.qiniu.io/ava/manual/4479/workspace).

## 查看训练监控
如果用户需要更为详细、定制化的训练监控选项，请点击[训练监控管理](05-tasks/5.2.3-monitor.md)查询.

# 训练产生的模型
对于训练产生的模型, 可以在 AVA 平台的模型模块->训练产生模型 tab 进行查看. 找到上面创建的训练"cifar10-demo”, 点击左边的展开加号, 就可以看到训练中指定保存的训练产生模型.
您可以下载满足您需求的模型.

![](https://odum9helk.qnssl.com/FgCh5xBtPChY2Swl5lDGF0SMkbsC?imageView2/2/w/800)

恭喜您, 至此, 您就在 AVA 平台上训练出了一个模型.



