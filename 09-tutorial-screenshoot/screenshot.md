# 视频截图在工作流中的实现
本实例中，将详细讲解如何通过使用七牛深度学习平台的[工作流](/05-tasks/5.6-workflow.md),实现将一个视频文件按需求截图并上传至七牛云存储Bucket中.基本操作请参考[5.6 工作流](/05-tasks/5.6-workflow.md).

在工作流中用户首先需要将复杂工作分解为由多个步骤组成的计算图, 每个步骤的任务由一个自定义镜像或脚本执行. 然后将每个步骤定义为一个任务模版, 并将任务模版组合成工作流模版. 之后即可根据需要将已定义的工作流模版实例化并执行工作流实例.

### 创建任务定义
根据需求，本实例可以分成4个任务：

1. download-video：将视频下载并重命名(videoname)后放入共享文件夹(dir)
2. ffmpeg：利用ffmpeg工具对视频截图，并存入准备上传的文件夹(src-dir)
3. get-upload-config：下载上传配置文件，并存入共享文件夹中(dir)
4. upload-screenshoots：将截图上传至指定的bucket中
* 任务定义所包含的输入参数及关系如下图所示：
![](/images/ch-09/workflow-structure.png)

**输入参数**
+ url：视频的url地址
+ videoname：视频名（输出的截图以 视频名-%d.JPG 命名）
+ dir：存放需要处理的视频及上传所需配置文件的目录（需在共享目录下）
+ fps：截图的频率（fps=1时为每一秒截一张图，fps=1/20是为每二十秒截一张图）
+ scr-dir：截图输出文件夹（需要在共享目录下）
+ AK：AccessKey（左侧边栏-个人中心-密钥管理中获得）
+ SK：SecretKey（左侧边栏-个人中心-密钥管理中获得）
+ username：账户的名字, 可以任意取，和在七牛注册的邮箱信息没有关系
+ bucket：七牛云存储空间名
+ configname：配置文件名

**镜像选择及执行入口**
+ download-video

    镜像地址：
    >reg.qiniu.com/common/public-mxnet:public-test

    执行入口：
    ```dockerfile
    wget {{inputs.parameters.url}} -O {{inputs.parameters.videoname}} && mv  {{inputs.parameters.videoname}} {{inputs.parameters.dir}}
    ```
+ ffmpeg
    镜像地址：
    >reg.qiniu.com/avatest/detect-reader:20181212-31fbdc6

    执行入口：
    ```dockerfile
    mkdir -p {{inputs.parameters.src-dir}} && cd {{inputs.parameters.dir}} && ffmpeg -i {{inputs.parameters.videoname}} -f image2 -vf fps=fps={{inputs.parameters.fps}} {{inputs.parameters.src-dir}}/{{inputs.parameters.videoname}}-%d.jpg
    ```
+ get-upload-conf

    镜像地址：
    >reg.qiniu.com/avatest/qshell:20181211-2020

    执行入口：
    
    通过qshell get获取配置文件（配置文件必须在bucket中）
    ```dockerfile
    qshell account {{inputs.parameters.ak}} {{inputs.parameters.sk}} {{inputs.parameters.username}} && qshell get {{inputs.parameters.bucket}} {{inputs.parameters.configname}} && mv {{inputs.parameters.configname}} {{inputs.parameters.dir}}
    ```

    通过wget获取配置文件（需要配置文件的url）
    ```dockerfile
    wget {{inputs.parameters.configurl}} -O {{inputs.parameters.configname}} && mv {{inputs.parameters.configname}} {{inputs.parameters.dir}}
    ```
+ upload-screenshoot

    镜像地址：
    >rreg.qiniu.com/avatest/qshell:20181211-2020

    执行入口：
    ```dockerfile
    cd {{inputs.parameters.dir}} && qshell account {{inputs.parameters.ak}} {{inputs.parameters.sk}} {{inputs.parameters.username}} && qshell qupload {{inputs.parameters.configname}}
    ```
    配置文件中的内容为：
     ```dockerfile
    {
        "src_dir" : "xxx"（与输入参数中的src-dir一致),
        "bucket" : "xxx"
    }
    ```
    详情可参考[qshell](https://developer.qiniu.com/kodo/tools/1302/qshell)及[qshell qupload](https://github.com/qiniu/qshell/blob/master/docs/qupload.md)

### 创建工作流模版
![](/images/ch-09/create-template.png)
工作流模版结构如上图所示，任务定义中的输入参数需要绑定全局参数。本实例中，对不同任务所声明的相同输入参数需要绑定相同的全局参数。

### 创建工作流
选择创建的模版

输入参数的取值如下图所示：
![](/images/ch-09/input.png)
为任务选择完资源后,即可创建并运行该工作流。

截图会上传至配置文件中所输入的bucket中，结果如下图所示：
![](/images/ch-09/workflow-result.png)



