原CSDN博客地址
http://blog.csdn.net/byhook/article/details/50168049

笔者生产环境是ubuntu14.04.2
一直都听说FFmpeg很强大很暴力
但一直都没时间研究沉淀
今天心血来潮,决定试试看
上正文
首先下载FFmpeg源代码
[官方地址](http://www.ffmpeg.org/releases/ffmpeg-2.8.3.tar.gz)
[Github地址](https://github.com/FFmpeg/FFmpeg.git)
1.首先修改configure文件

去除后缀名之后的版本号
#修改前

当然了可以写成一个Shell脚本librename.sh
```
dot="'"

SLIBNAME_WITH_MAJOR=(LIBNAME_WITH_MAJOR=$dot'$(SLIBNAME).$(LIBMAJOR)'$dot)
SLIBNAME_WITH_MAJOR_REP=(LIBNAME_WITH_MAJOR=$dot'$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'$dot)
SLIB_INSTALL_NAME=(SLIB_INSTALL_NAME=$dot'$(SLIBNAME_WITH_VERSION)'$dot)
SLIB_INSTALL_NAME_REP=(SLIB_INSTALL_NAME=$dot'$(SLIBNAME_WITH_MAJOR)'$dot)
SLIB_INSTALL_LINKS=(SLIB_INSTALL_LINKS=$dot'$(SLIBNAME_WITH_MAJOR)\s$(SLIBNAME)'$dot)
SLIB_INSTALL_LINKS_REP=(SLIB_INSTALL_LINKS=$dot'$(SLIBNAME)'$dot)

sed -i 's/'$SLIBNAME_WITH_MAJOR'/'$SLIBNAME_WITH_MAJOR_REP'/g' configure
sed -i 's/'$SLIB_INSTALL_NAME'/'$SLIB_INSTALL_NAME_REP'/g' configure
sed -i 's/'$SLIB_INSTALL_LINKS'/'$SLIB_INSTALL_LINKS_REP'/g' configure
```
2.编写脚本文件build.sh
```
NDK=/work/android-ndk-r10e
SYSROOT=$NDK/platforms/android-9/arch-arm/  
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64  
  
function build_one  
{  
  ./configure \
   --prefix=$PREFIX \
   --enable-shared \
   --disable-static \
   --disable-yasm \
   --disable-doc \
   --disable-ffserver \
   --enable-cross-compile \
   --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
   --target-os=linux \
   --arch=arm \
   --sysroot=$SYSROOT \
   --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
   --extra-ldflags="$ADDI_LDFLAGS" \
   $ADDITIONAL_CONFIGURE_FLAG
}  
CPU=arm  
PREFIX=~/ffmpeg/$CPU  
ADDI_CFLAGS="-marm"  
build_one  
```

注意NDK,SYSROOT,TOOLCHAIN换成自己本机的地址
添加build.sh的权限
```
chmod -R 777 build.sh
```
执行
```
./build.sh
```
如果出现
```
WARNING: /work/android-ndk-r10e/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64/bin/arm-linux-androideabi-pkg-config not found, library detection may fail.
```
可以忽略
然后执行
```
make
make install
```
接着在目录
```
~/ffmpeg/arm
```
就有我们需要的文件
```
drwxrwxr-x 2 byhook byhook 4096 12月  3 23:58 bin
drwxrwxr-x 9 byhook byhook 4096 12月  3 23:58 include
drwxrwxr-x 3 byhook byhook 4096 12月  3 23:58 lib
drwxrwxr-x 3 byhook byhook 4096 12月  3 23:58 share
```

最新编译方案
FFmpeg合并为一个库
[http://blog.csdn.net/byhook/article/details/51971652](http://blog.csdn.net/byhook/article/details/51971652)

参考
[http://blog.csdn.net/gobitan/article/details/22750719](http://blog.csdn.net/gobitan/article/details/22750719)
