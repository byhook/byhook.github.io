```
本机OS: Ubuntu 14.04 x64
```
首先上官方文档
https://developer.android.com/ndk/guides/standalone_toolchain.html
可以自定义工具链进行交叉编译

1.对不同的指令集APP_ABI设置

| `Architecture`           | `Toolchain name`  |
| :----------------- |:-----------------|
|`ARM-based`	|`arm-linux-androideabi-<gcc-version>`
|`x86-based`	|`x86-<gcc-version>`
|`MIPS-based`	|`mipsel-linux-android-<gcc-version>`
|`ARM64-based	` |`aarch64-linux-android-<gcc-version>`
|`X86-64-based`	 |`x86_64-<gcc-version>`
|`MIPS64-based`	|`mips64el-linux-android--<gcc-version>`



2.工具链和相应的值，使用--arch

| `Toolchain`           | `Value`  |
| :----------------- |:-----------------|
|`mips64 compiler`	|`--arch=mips64`
|`mips GCC 4.8 compiler`	|`--arch=mips`
|`x86 GCC 4.8 compiler`	|`--arch=x86`
|`x86_64 GCC 4.8 compiler`	|`--arch=x86_64`
|`mips GCC 4.8 compiler`	|`--arch=mips`

3.工具链和相应的值，使用--toolchain

| `Toolchain`           | `Value`  |
| :----------------- |:-----------------|
|`arm`|`--toolchain=arm-linux-androideabi-4.8`|
|`arm`|`--toolchain=arm-linux-androideabi-4.9`|
|`arm`|`--toolchain=arm-linux-android-clang3.5`|
|`arm`|`--toolchain=arm-linux-android-clang3.6`|
|`x86	`|`--toolchain=x86-linux-android-4.8`|
|`x86	`|`--toolchain=x86-linux-android-4.9`|
|`x86	`|`--toolchain=x86-linux-android-clang3.5`|
|`x86	`|`--toolchain=x86-linux-android-clang3.6`|
|`mips`|`--toolchain=mips-linux-android-4.8`|
|`mips`|`--toolchain=mips-linux-android-4.9`|
|`mips`|`--toolchain=mips-linux-android-clang3.5`|
|`mips`|`--toolchain=mips-linux-android-clang3.6`|
|`arm64`|`--toolchain=aarch64-linux-android-4.9`|
|`arm64`|`--toolchain=aarch64-linux-android-clang3.5`|
|`arm64`|`--toolchain=aarch64-linux-android-clang3.6`|
|`x86_64`|`--toolchain=x86_64-linux-android-4.9`|
|`x86_64`|`--toolchain=x86_64-linux-android-clang3.5`|
|`x86_64`|`--toolchain=x86_64-linux-android-clang3.6`|
|`mips64`|`	--toolchain=mips64el-linux-android-4.9`|
|`mips64`|	`--toolchain=mips64el-linux-android-clang3.5`|
|`mips64`|	`--toolchain=mips64el-linux-android-clang3.6`|

主机工具链和相应的值，使用-system

 |`Host toolchain ` |`Value` |
| :----------------- |:-----------------|
|`64-bit Linux`|`-system=linux-x86_64`|
|`64-bit MacOSX`|`-system=darwin-x86_64`|
|`64-bit Windows`|`-system=windows-x86_64`|

自定义
```
#NDK_HOME为安装路径
export NDK_HOME=/workspace/android-ndk-r10e
$NDK_HOME/build/tools/make-standalone-toolchain.sh --arch=arm --platform=android-21 --install-dir=$HOME/android-toolchain --toolchain=arm-linux-androideabi-4.9
```
上面演示的仅仅是单一的arm工具链
可以根据自己的需要独立配置
不过相应的arch和对应的toolchain要对应

可以写个Shell脚本处理make_toolchain.sh
在开头配置好相应的路径，和platform即可
```

export NDK_HOME=/workspace/android-ndk-r10e


platform=android-21
shmake=$NDK_HOME/build/tools/make-standalone-toolchain.sh

archs=(
    'arm'
    'arm64'
    'x86'
    'x86_64'
    'mips'
    'mips64'
)

toolchains=(
    'arm-linux-androideabi-4.9'
    'aarch64-linux-android-4.9'
    'x86-4.9'
    'x86_64-4.9'
    'mipsel-linux-android-4.9'
    'mips64el-linux-android-4.9'
)

echo $NDK_HOME
num=${#archs[@]}
for ((i=0;i<$num;i++))
do
   sh $shmake --arch=${archs[i]} --platform=$platform --install-dir=$HOME/android-toolchain/${archs[i]} --toolchain=${toolchains[i]}
done
```
运行
```
sh make_toolchain.sh
```

![](http://upload-images.jianshu.io/upload_images/2006464-860cd25dfde40761.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2006464-37a4d2b0311f0997.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

交叉编译的工具链配置完成，方便后续进行交叉编译
