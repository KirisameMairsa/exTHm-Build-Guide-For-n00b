## 什么是 exTHmUI ？

![exthm logo](https://i.niupic.com/images/2020/06/16/8gWB.png)

这是一个由国内东方众自行发起制作、以东方 Project 元素为特色的类原生 Android 项目，基于 [LineageOS](https://github.com/LineageOS)

> exthm-10 EOL 时间为2021年底，届时请迁移至本exthm-11分支
> 很抱歉让大家失望了qwq
> 因为一些原因我们未能实现当初的基于AOSP的想法，因此本11分支将继续基于LineageOS
> AOSP基线为android-11.0.0_r29,LineageOS基线为18.1

> 本页面仅供想要成为 Build Bot 的萌新阅读：）各位大佬请多多包含 xD
>
> 这是一个包教包会的 Build Bot 教学，尽管这样做非常不好

## 如何编译 exTHmUI ？
### 编译环境

#### <!--注意：优先推荐使用服务器进行编译，以节省时间，提高效率。-->

#### 参考 Android Open Source Project 的[官方硬件要求](https://source.android.com/setup/build/requirements)，我们给出以下最低配置要求：

- 操作系统：使用 64 位环境的 Linux 操作系统（如果是不了解 Linux 的新手，优先推荐使用 Ubuntu 20.04 LTS）
- 至少需要 16 GB 的可用 RAM
- 最好使用能够流畅访问 github.com 与 source.android.com 的网络

### 安装相关依赖

#### Debian系（以Ubuntu为例）

```shell
sudo apt-get install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev python
```

- 低于 20.04 的 Ubuntu，还需安装：`libwxgtk3.0-dev`
- 低于 16.04 的 Ubuntu，还需安装：`libwxgtk2.8-dev`

##### 安装 repo

```shell
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
sudo cp ~/bin/repo /bin/repo
sudo chmod a+x /bin/repo
```

##### 如果无法访问 Google ，可以使用 TUNA 的 repo 镜像

```shell
mkdir -p ~/bin
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo > ~/bin/repo
sudo cp ~/bin/repo /bin/repo
sudo chmod a+x /bin/repo
```

#### Arch系（Archlinux/Manjaro等）

```
cd ~/
git clone https://github.com/akhilnarang/scripts
cd scripts
./setup/arch-manjaro.sh
```

### 修改AOSP为国内源（可选）

```
git config --global url.https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/.insteadof https://android.googlesource.com

repo的运行过程中会尝试访问官方的git源更新自己，如果想使用tuna的镜像源进行更新，可以将如下内容复制到你的~/.bashrc里或者是直接运行
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```
> 或者是修改.repo/manifests.git/config内的
```
url = https://android.googlesource.com/platform/manifes
改为
url = https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest
```
### 设置 Git 用户名及邮箱（替换成自己的）

```shell
git config --global user.email "xxx@xxx.com"
git config --global user.name "xxx"
```

### 新建工作区

> 这里的工作区指存放 exTHmUI 源码的路径

```shell
# 新建文件夹用来存放 exTHmUI 源码
# ~/指用户目录
# 用户目录指 /home/用户名
# 此处的 exTHmUI 为文件夹名，可自定义
mkdir ~/exTHmUI
cd ~/exTHmUI
```

### 初始化仓库

> repo 是一个用来管理多个仓库的工具

要开始编译 exTHmUI，你需要熟悉 Repo 和 Git

要开始同步源码，请使用这些命令

```shell
repo init -u https://github.com/exthmui/android.git -b exthm-11
```

您也可以使用如下命令来初始化仓库，这样可以节省磁盘空间

```shell
# depth=1 表示不同步历史提交
repo init -u https://github.com/exthmui/android.git -b exthm-11 --depth=1
```

### 同步源码

> [方括号]内为可选参数，可以根据网络情况按需使用。

```shell
# -j 参数为下载线程数，不带该参数默认自动分配线程
# -l 参数用于更新本地存储库
# --fail-fast 参数可以处理上次repo时由于网络原因导致的同步失败问题
#--force-sync 参数忽略本地修改，强制同步git仓库的内容。
repo sync [-j8 --fail-fast --force-sync]
参考同步命令
repo sync -j$(nproc --all) -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

### 同步设备树

> 请尊重开发者们的劳动成果，如果使用了其他开发者开源的设备树或者内核，请声明出处
>
> 请勿做出剽窃，抄袭，倒卖，诽谤开发者等下流行为
>
> 我们不主张过多的去编译固件，Build Bot 不等于开发者，按照本文的流程最多只能算作一个 Build Bot，本文的全部流程实际上使用脚本就可以完成。
>
> 请尊重开发者们的劳动成果，并且努力成为一名真正的开发者。

##### 寻找编译机型的设备树(Device)，内核源码(Kernel)以及供应商文件（Vendor)：

<!--通常情况下，建议使用同一个作者的设备树，内核源码以及供应商文件-->

<!--部分设备可能并不需要同步通用（common)设备树，供应商文件，如果使用预编译内核，则无需同步内核源码-->

以 OnePlus 8(设备代号 instantnoodle)为例，如果我们想要基于现有的开发者们的劳动成果去编译 exTHm，步骤如下

笔者选择使用 Lineage OS 仓库中的开源内核为例

首先我们去 Lineage OS 的仓库搜索 instantnoodle，可以得到 `oneplus_device_instantnoodle`，这就是我们的设备树

因为 OnePlus 8 使用的 CPU 为 sm8250，所以我们可以搜索到`kernel_oneplus_sm8250` 与 `device_oneplus_sm8250-common`

通常我们可以在设备树中找到 dependencies 文件，它会告诉我们需要同步哪些文件

供应商文件？如果你搜索设备代号无果，可以在 [TheMuppets 的仓库](https://github.com/TheMuppets) 寻找对应的厂商 

依旧是以 OnePlus 8 为例，我们可以在 proprietary_vendor_oneplus 中同步得到需要的厂商文件`sm8250-common` 与 `instantnoodle`

##### exTHm-ify 

请参考此提交：[vince: exthm-ify · exthmui-devices/android_device_xiaomi_vince@03b4077 (github.com)](https://github.com/exthmui-devices/android_device_xiaomi_vince/commit/03b4077f3c74daf78d0ca0da0a9fccbb39c89bd4)

### 开始编译

```shell
. build/envsetup.sh
lunch exthm_[您的设备代号]-userdebug
mka bacon
```

## 查阅 wiki

如果对详细的系统介绍和编译有兴趣，欢迎访问我们的 [Wiki](https://wiki.exthmui.cn/)
=======



## 向 exTHmUI 贡献提交

我们欢迎开发者通过 [Gerrit](https://www.gerritcodereview.com/) 参与到 exTHmUI 的开发中，请访问 [exTHmUI Code Review](https://review.exthmui.cn/)



```

### 提交commit
> 我们欢迎开发者在Gerrit提交commit，你可以使用以下命令提交commit：
```
    cd <project>
    <make edits>
    git add -A
    git commit -m "commit信息"
    git push ssh://<username>@review.exthmui.cn:29418/<project> HEAD:refs/for/exthm-11
```
# 如何使用SSH提交到Gerrit
> 首先生成SSH密钥
```
     git config --global user.email "xxx@xxx.com"
     git config --global user.name "xxx"
     ssh-keygen -t rsa
     然后后面的提示一路回车即可

```
> 复制密钥
```
     cat ~/.ssh/id_rsa.pub

```

>配置ssh key
![Gerrit ssh](https://img-blog.csdnimg.cn/20181106150940882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pob3Vsd18yNQ==,size_16,color_FFFFFF,t_70)
```
     首先打开exTHmUI代码审查(下面有) -> 点击右上角的Sign in -> 再点击Github OAuth2那一行使用git登录 
     然后点击右上角的设置 (齿轮图标)
    
     1.找到Email Addresses部分 添加您的邮箱
    
     2.在您的邮件收件箱找到Gerrit的验证邮件, 点击它通过验证
    
     3.转到SSH Keys部分将密钥输出内容粘贴到SSH Keys中的New SSH Keys输入框中
    
     4.点击添加按钮 (即 ADD NEW SSH KEY), 然后点击SAVE CHANGES
    
     5.稍后您的邮件中收到Gerrit的答复.
    
     好耶!!

```


>推送提交至审核服务器

所需文件: [commit-msg](https://review.exthmui.cn/tools/hooks/commit-msg)

```
     1. 首先进入到带有您要推送更改提交的源码目录
    
     2. 添加远程服务器 输入git remote add gerrit <审核服务器地址>/源码目录的仓库名称
         < 例如 git remote add gerrit https://review.exthmui.cn/android >
    
     3. 设置Change-Id功能
         3-1: 下载commit-msg文件 并将它移动至 <源码目录/.git/hooks> 下
         3-2: 输入 chmod a+x .git/hooks/commit-msg
     
     4. 将您要修改好的文件添加到暂存区, 并写好提交描述
         * 如果先前修改好的提交可以使用 git commit --amend 重新提交
    
     5. 然后将提交推送到审核服务器
         即 git push ssh://<用户名>@<代码审查服务器地址>:<端口>/<仓库名称> HEAD:refs/for/<分支>
         * 例如 < git push ssh://builder@review.exthmui.cn:29418/android HEAD:refs/for/exthm-11 >
    
     6. 出现 SUCCESS 即推送完成, 待审查人与审稿人测试无误后即可合并
    
     好耶!!

```


> 一次提交即可提交commit。使用以下命令压缩多个提交：`git rebase -i HEAD~<提交数量>`

> 如果您要添加其他内容，只需重复步骤（不要提交新的commit），而是使用`git commit --amend`代替`git commit -m`。Gerrit会将其识别为新的commit集。

> 要查看您和其他commit的状态，请访问[exTHmUI](https://review.exthmui.cn)代码审查。
