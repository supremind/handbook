# 在AVA平台跑OCR训练

## 在工作台中跑OCR训练

工作台有两种类型，CPU工作台和GPU工作台，GPU工作台一般用来做调试用，但也可以跑训练。下面我们展示如何用GPU工作台跑OCR训练。值得注意的是，这里在工作台跑训练只是为了让demo过程更加清晰，实际上更多应该起训练任务来运行训练。

### 环境准备

1. 创建AVA工作台

   第一步：从工作台标签进入，选择“新建工作台”，输入名称"ocr_test"
   ![](/images/ch-07/create_workspace.png)

   第二步：选择数据集，跳过这一步

   第三步：选择镜像。我们选择ava-mxnet的py35-cuda80-cudnn7-eval镜像

   ![](/images/ch-07/ava_image.png)

   第四步：选择计算资源，这里选择使用GPU，比如选择K80，然后选择4核CPU，60G内存

   ![](/images/ch-07/ava_resource.png)

   第五步：选择“创建”，等待几秒钟，工作台可创建出来了

2. MXNET Example代码

我们采用MXNet框架来跑OCR训练。MXNET自带缺省的OCR训练例子在example/ctc目录中，我们可以简单的用如下git命令clone mxnet：

git clone git@github.com:apache/incubator-mxnet.git

然后进入incubator-mxnet目录，把example目录拷贝的自己 PVC共享目录。加入你的组名叫group1，你自己的id为avatest，那么，把example目录拷贝到/workspace/mnt/group/group1/avatest下面

3. 安装需要的软件依赖包。这里，我们不准备用WARPCTC，那样需要从源码重新安装MXNET（详情见：https://github.com/apache/incubator-mxnet/tree/master/example/ctc或者example/ctc/README.md）

   然后安装captch:  pip3 install captcha

   接着安装numpy：pip3 install -U numpy

   好了，由于ava的镜像安装了CUDA及相关的依赖，这里的OCR训练只需要安装captcha和NUMPY就可以开始训练了。

### 启动训练任务跑OCR训练

现在进入工作台，启动一个Shell，然后进入example/ctc目录下面，ls一下，可以看到：

​	![](/images/ch-07/ls.png)

其中，myinfer.sh是批量脚本，后面跑训练的时候用的，这里暂时忽略它。

下面演示如何启动训练，再用推理认识图片中的文字。

第一步：准备字体文件。我们使用的是Ubuntu16.04操作系统，字体文件存放在/usr/share/font下面，我们随便采取一个，拷贝到当前目录，执行如下命令：

cp /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf .

第二步：准备样本图片，训练结束后会识别该样本图片中的文字。执行下面命令生成样本图片：

python3 captcha_generator.py DejaVuSans.ttf sample1.jpg

从Jupyter的Files tab中可以看到，生成的sample包含字符8405：

![](/images/ch-07/sample.png)

第三步：启动训练。由于这次工作台采用GPU，故可以采用GPU模式训练：

python3 lstm_ocr_train.py --gpu 2 --num_proc 2 --loss ctc DejaVuSans.ttf

命令中，—gpu 2指定用两张GPU卡训练，刚好本次工作台创建的时候使用了两张GPU卡。—num_proc表示同时启动多少个线程跑训练，一般设置和GPU数目一样。—loss ctc表示采取ctc loss模式。

第四步：观察训练过程。这次训练会跑100个epoch，刚开始的时候，accuracy都是0，这个是正常现象：

![](/images/ch-07/train_start.png)

到20个epoch左右，accuracy显著提升，最后可以提升到0.95以上的准确率：

​					![](/images/ch-07/train_end.png)

第五步：推理。训练结束后，文件夹中生成很多模型文件，一般是一个epoch一个，我们准备采用第100个epoch生成的模型做推理：

​	![](/images/ch-07/output.png)

推理使用的命令是：python3 lstm_ocr_infer.py --prefix ocr --epoch 100 sample1.jpg

推理结束后，会在终端输出：Digits: [8, 4, 0, 5]。

## 启动训练任务执行训练

上面演示了如何通过工作台运行一个OCR操作，下面演示下如何启动训练来完成相同的过程。对于本例子来讲，两个过程做的事情是一样的，起训练任务是把在工作台中做的事情自动做了批量处理。

由于训练需要一个可执行文件作为入口，这里我们编辑一个shell脚本，存放在myinfer.sh中：

cd /workspace/mnt/group/ava1/avatest/example/ctc
pip3 install captcha
pip3 install numpy
cp /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf .
python3 captcha_generator.py DejaVuSans.ttf sample1.jpg
python3 lstm_ocr_train.py --gpu 2 --num_proc 2 --loss ctc DejaVuSans.ttf
python3 lstm_ocr_infer.py --prefix ocr --epoch 100 sample1.jpg

需要注意的是，需要把上面的avatest改成你自己的名字，然后确保系统中存在DejaVuSan.ttf字体文件。如果没有这个字体文件，选用其他的也可以。