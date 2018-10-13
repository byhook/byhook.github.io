**目录**
- [1.简介](#1.简介)
- [2.官方同步源代码](#2.官方同步源代码)
- [3.镜像同步源代码](#3.镜像同步源代码)
- [4.已有源代码更新](#4.镜像同步源代码)
- [5.编译源代码](#5.编译源代码)
   - [5.1编译Android 4.1.2](#5.1编译Android 4.1.2)
   - [5.2编译Android 5.1.1](#5.1编译Android 5.1.1)
- [6.参考](#6.参考)

###1.简介
之前二次开发Launcher的时候有同步过官方Android 4.1.2的源代码，遗憾当时没有记录下载过程，现在重新温习一下，其实也比较简单。
###2.官方同步源代码
[官网网址](https://source.android.com/source/downloading.html) 需要翻墙
2.1新建一个用于存放下载脚本文件的目录
```
mkdir ~/bin
PATH=~/bin:$PATH
```
2.2下载Repo脚本文件
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
2.3创建用于存放Android源代码的目录
```
mkdir android_source
cd android_source
```
2.4初始化
```
repo init -u https://android.googlesource.com/platform/manifest
```
上述命令会要求Repo下载最新的Android源代码，也就是master分支，如果想下载其他分支
```
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```
2.5同步Android源代码
```
repo sync
```
下载过程保持网络通畅，笔者网络较慢，同步了快一整天。

###3.镜像同步源代码
- 对于没有翻墙的用户，可以使用清华大学的镜像。
https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/
3.1.1 同上述步骤，新建一个用于存放下载脚本文件的目录
```
mkdir ~/bin
PATH=~/bin:$PATH
```
3.1.2 下载Repo
```
git clone https://aosp.tuna.tsinghua.edu.cn/android/git-repo.git/
cp git-repo/repo ~/bin/
```
3.1.3 修改Repo文件
～/bin/repo
```
REPO_URL = 'https://gerrit.googlesource.com/git-repo'
改为
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'
```
3.1.4 创建用于存放Android源代码的目录
```
mkdir android_source
cd android_source
```
3.1.5 同步源代码
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
repo sync -j4
```

- 由于首次同步需要下载 24GB 数据，过程中任何网络故障都可能造成同步失败，建议直接使用初始化包进行初始化。
3.2.1下载初始包
```
#下载重试不限次数，防止网络异常中断
wget -c -t 0 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 
tar -vxzf aosp-latest.tar
cd aosp 
#这时ls的话什么也看不到，因为只有一个隐藏的.repo目录
repo sync
```
3.2.2选择版本同步
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
repo sync -j4
```
下载好了就是下图
![](http://upload-images.jianshu.io/upload_images/2006464-2c5ede513a9c4688.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

源代码目录含义:

|目录名|描述|
|:---|:---|
|abi|应用程序二进制接口|
|bionic|C/C++运行时库，在NDK程序中很大一部分调用就是这里的程序|
|bootable|用于Android装载和启动程序，其中就包括bootloader和recovery。bootloader是Android中唯一在LInux内核之前执行的程序。通过这段程序可以初始化硬件，建立内存控件的映射图等，总之，bootloader就是为LInux内核准备合适的运行环境。|
|build|用于编译Android源代码以及建议system.img，ramdisk.img等文件的工具|
|cts|用于兼容性测试的工具|
|dalvik|Dalvik虚拟机的源代码|
|development|高层的开发和调试工具|
|device|与设备相关的代码|
|docs|包含与Android源代码项目的文档和工具，如Dalvik虚拟机格式文档等|
|external|扩展工具的源代码|
|framworks|Android框架层源代码。也就是Android SDK的源代码|
|hardware|硬件层接口和库|
|libcore|Java核心库|
|ndk|NDK相关的源代码|
|packages|与Android系统一同发布的应用程序的源代码|
|prebuilts|Android在各种平台下编译之前要使用的工具|
|sdk|在开发环境中使用的工具，如ddms，draw9path，sdkmanager等|
|system|Android的基本系统|

注意：
查看所有分支
```
cd .repo/manifests
git branch -a
```

![](http://upload-images.jianshu.io/upload_images/2006464-fa79346fef0c10d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果想切换到其它分支可以重新执行
```
repo init -b android-6.0.1_r63
repo sync
```

###4.已有源代码更新
如果手上已有Android系统源代码，可以通过代理远程更新，不过速度可能会比较慢。另外还可以
4.1修改~/bin/repo文件
```
REPO_URL = 'https://gerrit.googlesource.com/git-repo'
改为
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'
```
4.2修改源代码目录.repo/manifests.git/config
```
url = https://android.googlesource.com/platform/manifest
改为
url = https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```
4.3修改源代码目录.repo/manifest.xml
```
<manifest>

   <remote  name="aosp"
-           fetch="https://android.googlesource.com"
+           fetch="https://aosp.tuna.tsinghua.edu.cn"
            review="android-review.googlesource.com" />

   <remote  name="github"
```
最后直接同步即可
```
repo sync -j4
```
###5.1编译Android 4.1.2
笔者下载的是Android 4.1.2源代码。
默认的源代码仅能在64位机器上编译。编译过程有很多坑，要有心里准备。
5.1.1进入源码目录初始化编译环境
```
source build/envsetup.sh
```
5.1.2选择目标
```
lunch full-eng
```
设置编译目标为full-eng，表示正对所有的移动设备，Android模拟器有效，并打开所有的调试选项。
只执行lunch命令，会出现对应的选项

![](http://upload-images.jianshu.io/upload_images/2006464-20f3aad2fd7f059a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.1.3编译Android源代码
make命令只会利用一个CPU核进行编译，如果是多核CPU，为了缩短时间，可以加上-jn参数。
注意:
```
#Android 4.1.2源代码编译要求
笔记本OS:  Ubuntu 16.04 x64
GNU Make版本: 3.8.1或者3.8.2(笔者用的是3.8.2)
JDK 版本: JDK 1.6
```
编译过程你很可能会碰到如下问题
```
1. /bin/bash: xsltproc: 未找到命令
2. /bin/bash: flex: 未找到命令
3. /bin/bash: bison: 未找到命令
4. sh: 1: gperf: not found
5. /bin/bash: xmllint: 未找到命令
6. failed--compilation aborted at external/webkit/Source/WebCore/make-hash-tools.pl line 2
7. /usr/include/stdlib.h:759:34: fatal error: bits/stdlib-bsearch.h: No such file or directory
```
建议提前安装好下列必要依赖
```
sudo apt-get install xsltproc flex bison gperf libxml2-utils libswitch-perl gcc-multilib
```
最后开始编译
```
make -j4
```
笔者笔记本编译花了接近3个小时

![](http://upload-images.jianshu.io/upload_images/2006464-a3547c3010b07b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###5.2编译Android 5.1.1
源代码的下载参考上述步骤
环境配置
```
#公司的台式机
本机OS:  Ubuntu 14.04 x64
JDK版本:  openjdk 1.7
```
配置过程同上，编译过程中如果出现
```
You asked for an OpenJDK 7 build but your version is
java version "1.7.0_79" Java(TM) SE Runtime Environment (build 1.7.0_79-b15) Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode).
```
建议更换JDK为openjdk 1.7
```
apt-get install openjdk-7-jdk
```
如果出现
```
/bin/bash: build/tools/findleaves.py: 权限不够
```
直接给文件加上执行权限，笔者是直接在源码目录
```
chmod a+x -R *
```
笔者编译完大概也是3个多小时，过程跟4.12编译差不多
![](http://upload-images.jianshu.io/upload_images/2006464-3007e2b9818aba14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动模拟器
```
emulator &
```
如图

![](http://upload-images.jianshu.io/upload_images/2006464-7eb53046b83ad57f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意：几个很有用的命令。
```
make -k 继续编译
m  编译完整的Android源代码
mm  此命令必须进入指定的工程的目录进行编译
mmm  可以在任何一级目录编译任意指定的工程
```

注意：
同步出现如下错误
```
error: .repo/repo/: contains uncommitted changes
```
进入相关目录commit一次，然后pull更新一下即可。

###6.参考
https://source.android.com/source/downloading.html
https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/
http://blog.csdn.net/michaelpp/article/details/22801953
http://blog.csdn.net/ambulong/article/details/51627115
《Android 深度探索(卷1)：HAL与驱动开发》