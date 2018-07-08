# 训练一个 OCR 模型

## 在工作台中执行 OCR 训练

工作台有两种类型: CPU 工作台和 GPU 工作台. GPU 工作台一般做调试用, 但也可以运行训练, 方便调试训练代码. 下面我们展示如何用 GPU 工作台运行 OCR 训练. 值得注意的是, 这里在工作台运行训练只是为了让 demo 过程更加清晰, 实际上应该启动训练任务来运行训练.

### 环境准备

#### 创建AVA工作台

1. 从工作台标签进入, 选择 "新建工作台"

    输入名称"ocr_test"

    ![](/images/ch-07/create_workspace.png)

2. 选择数据集

    跳过这一步

3. 选择镜像

    我们选择 `ava-mxnet:py35-cuda80-cudnn7` 镜像

    ![](/images/ch-07/ava_image.png)

4. 选择计算资源

    这里选择使用 GPU, 然后选择 4 核 CPU, 60G 内存

    ![](/images/ch-07/ava_resource.png)

5. 点击 "创建", 等待几秒钟, 工作台可创建出来了

#### 下载 MXNet Example 代码

我们采用 MXNet 框架来运行 OCR 训练. MXNet 缺省包含的 OCR 训练例子在 `example/ctc` 目录中, 我们可以用如下 git 命令 clone MXNet 源代码: 

```bash
$ git clone git@github.com:apache/incubator-mxnet.git
$ cd incubator-mxnet
$ cp -r example /workspace/mnt/group/<your-group>/<your-name>/.
```

把 `example` 目录拷贝的自己的共享目录, 方便运行.

#### 安装依赖

这里, 我们不准备用 WARPCTC, 那样需要从源码重新安装 MXNet ([参考这里](https://github.com/apache/incubator-mxnet/tree/master/example/ctc))

- 安装 captch:  
    ```bash
    $ pip3 install captcha
    ```

好了, 由于 AVA 的镜像已经安装了 CUDA 及相关的依赖, 这里的 OCR 训练只需要安装 captcha 就可以开始训练了. 

### 运行训练

现在进入工作台, 启动一个 Terminal, 进入 `example/ctc` 目录下面, 可以看到:

​![](/images/ch-07/ls.png)

其中, `myinfer.sh` 是完整训练脚本, 后面运行训练的时候会用到, 这里暂时忽略它. 

下面演示如何启动训练, 再用推理识别图片中的文字. 

1. 准备字体文件

    我们使用的是 Ubuntu 16.04 操作系统, 字体文件存放在 `/usr/share/font` 下面, 我们可以使用任意字体, 拷贝到当前目录, 执行如下命令: 

    ```bash
    $ cp /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf .
    ```

2. 准备样本图片

    训练出的模型可用于识别图片中的文字. 执行下面命令生成样本图片: 

    ```bash
    $ python3 captcha_generator.py DejaVuSans.ttf sample.jpg
    ```

    点击 Jupyter Lab 的 Files tab 中可以看到, 生成的 sample 包含字符 8405: 

    ![](/images/ch-07/sample.png)

3. 启动训练

    由于这个工作台启用了 GPU, 可以采用 GPU 模式训练: 

    ```bash
    $ python3 lstm_ocr_train.py --gpu 2 --num_proc 2 --loss ctc DejaVuSans.ttf
    ```

    命令中, `—gpu 2` 指定用两张 GPU 卡训练, `—num_proc 2` 表示同时启动 2 个线程训练, 一般设置和 GPU 数目一样, `—loss ctc` 表示采取 ctc loss 模式. 

4. 查看训练过程

    这次训练会运行 100 个 epoch, 刚开始的时候, accuracy 都是 0: 

    ![](/images/ch-07/train_start.png)

    到20个epoch左右, accuracy 显著提升, 最后可以提升到 0.95 以上: 

    ![](/images/ch-07/train_end.png)

5. 推理

    训练结束后, 当前目录生成很多模型文件, 一般一个 epoch 一个文件, 我们采用第 100 个 epoch 生成的模型做推理:

    ![](/images/ch-07/output.png)

    执行命令: 
    ```bash
    python3 lstm_ocr_infer.py --prefix ocr --epoch 100 sample1.jpg
    ```

    输出: `Digits: [8, 4, 0, 5]`, 即为验证样本图中的字符.


## 启动训练任务执行训练

上面演示了如何通过工作台运行一个 OCR 训练, 下面演示如何启动 AVA 训练来完成相同的过程. 两种方式做的事情是一样的, AVA 训练专用于执行上述训练流程. 

1. 首先我们创建 shell 脚本 `myinfer.sh` 用于启动训练:

    ```bash
    cd /workspace/mnt/group/<your-group>/<your-name>/example/ctc
    pip3 install captcha
    pip3 install numpy
    cp /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf .
    python3 captcha_generator.py DejaVuSans.ttf sample1.jpg
    python3 lstm_ocr_train.py --gpu 2 --num_proc 2 --loss ctc DejaVuSans.ttf
    python3 lstm_ocr_infer.py --prefix ocr --epoch 100 sample1.jpg
    ```

    > 注意: 
    > 1. 需要把上面的 `<your-group>` 和 `<your-name>` 改成你自己的名字, 然后确保系统中存在 DejaVuSan.ttf 字体文件, 或更换其它字体文件
    > 2. `myinfer.sh` 文件需要有可执行权限

2. 创建训练与创建工作台流程类似, 不过需要为训练指定一个可执行入口, 这里指定为 `/workspace/mnt/group/<your-group>/<your-name>/example/myinfer.sh`

3. 启动训练

4. 在工作台中查看训练输出 log

    ```bash
    $ cd /workspace/mnt/group/<your-group>/<your-name>/example/run
    $ tail -f train-<your-train-name>-log
    ```

详细步骤可查阅[训练管理](/05-tasks/5.3-training.md)



