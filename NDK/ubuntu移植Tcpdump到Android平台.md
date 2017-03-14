```
系统: ubuntu14.04.2
NDK: android-ndk-r10e
```

**为Linux平台编译Tcpdump**

源码准备
[Tcpdump-4.7.4](http://www.tcpdump.org/release/tcpdump-4.7.4.tar.gz)
[libpcap-1.7.4](http://www.tcpdump.org/release/libpcap-1.7.4.tar.gz)

编译之前确保有lex和yacc工具
```
sudo apt-get install flex bison
```
1.解压libpcap-1.7.4之后进入该目录，打开终端
![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-f479131e4a5387b1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接着输入
```
make
```
完成编译
2.解压Tcpdump-4.7.4之后进入该目录，打开终端
同样

```
./configure
make
```
不出意外的话
![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-1ce992b6504c7c93?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看文件信息

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-e331c032b7767ece?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接输入

```
./tcpdump
```
可能会提示
![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-0bbaf56657a1b3ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要带上权限
```
sudo ./tcpdump
```


**为Android平台编译Tcpdump**

新建一个Shell脚本
笔者稍微做了一点修改
```
#!/bin/sh

# --------------------------------------
#
#      Title: build-tcpdump
#     Author: Loic Poulain, loic.poulain@gmail.com
# Updated by: muzso (http://muzso.hu/)
#
#    Purpose: download & build tcpdump for arm android platform
#
# You have to define your android NDK directory before calling this script
# example:
# $ export NDK=/home/Workspace/android-ndk-r10e
# $ sh build-tcpdump
#
# works with
# 	tcpdump 4.7.4
# 	android-ndk-r10e
#
# You'll need lex and yacc.
# On Debian/Ubuntu based systems run this:
#   sudo apt-get install flex bison
# --------------------------------------

# default, edit version 
tcpdump_ver=4.7.4
libpcap_ver=1.7.4
# note: libpcap v1.7.2 only required api v9, but libpcap v1.7.3+ requires api v21
# And tcpdump v4.7.4 requires libpcap v1.7.3+ too (tcpdump v4.7.3 could be compiled with libpcap v1.7.2).
# So viable combos are:
#   * api=9, libpcap=1.7.2, tcpdump=4.7.3
#   * api=21, libpcap=1.7.4, tcpdump=4.7.4
android_api_def=21
ndk_dir_def=android-ndk-r10e

#指定平台arm mips aarch64
platform=arm

#-------------------------------------------------------#

tcpdump_dir=tcpdump-${tcpdump_ver}
libpcap_dir=libpcap-${libpcap_ver}

if [ ${NDK} ]
then
	ndk_dir=${NDK}
else
	ndk_dir=${ndk_dir_def}
fi

ndk_dir=`readlink -f ${ndk_dir}`

if [ ${ANDROID_API} ]
then
	android_api=${ANDROID_API}
else
	android_api=${android_api_def}
fi

echo "_______________________"
echo ""
echo "NDK - ${ndk_dir}"
echo "Android API: ${android_api}"
echo "_______________________"


exit_error()
{
	echo " _______"
	echo "|       |"
	echo "| ERROR |"
	echo "|_______|"
	exit 1
}

{
	if [ $# -ne 0 ]
	then
		if [ -d $1 ]
		then
			cd $1
		else
			echo directory $1 not found
			exit_error
		fi
	else
		mkdir tcpdumpbuild
		cd tcpdumpbuild
	fi
}



# create env
{
	echo " ____________________"
	echo "|                    |"
	echo "| CREATING TOOLCHAIN |"
	echo "|____________________|"

	if [ -d toolchain ]
	then
		echo Toolchain already exist! Nothing to do.
	else
		echo Creating toolchain...
		mkdir toolchain
		bash ${ndk_dir}/build/tools/make-standalone-toolchain.sh --arch=$platform --platform=android-${android_api} --install-dir=toolchain
		
		if [ $? -ne 0 ]
		then
			rm -fr toolchain
			exit_error
		fi
	fi

	export CC=arm-linux-androideabi-gcc
	export RANLIB=arm-linux-androideabi-ranlib
	export AR=arm-linux-androideabi-ar
	export LD=arm-linux-androideabi-ld
	export PATH=`pwd`/toolchain/bin:$PATH
}

# download & untar libpcap + tcpdump
{
	echo " _______________________________"
	echo "|                               |"
	echo "| DOWNLOADING LIBPCAP & TCPDUMP |"
	echo "|_______________________________|"
	
	tcpdump_file=${tcpdump_dir}.tar.gz
	libpcap_file=${libpcap_dir}.tar.gz
	tcpdump_link=http://www.tcpdump.org/release/${tcpdump_file}
	libpcap_link=http://www.tcpdump.org/release/${libpcap_file}
	
	if [ -f ${tcpdump_file} ]
	then
		echo ${tcpdump_file} already downloaded! Nothing to do.
	else
		echo Download ${tcpdump_file}...
		wget ${tcpdump_link}
		if [ ! -f ${tcpdump_file} ]
		then
			exit_error
		fi
	fi
	
	if [ -f ${libpcap_file} ]
	then
		echo ${libpcap_file} already downloaded! Nothing to do.
	else
		echo Download ${libpcap_file}...
		wget ${libpcap_link}
		if [ ! -f ${libpcap_file} ]
		then
			exit_error
		fi
	fi
	
	if [ -d ${tcpdump_dir} ]
	then
		echo ${tcpdump_dir} directory already exist! Nothing to do.
	else
		echo untar ${tcpdump_file}
		tar -zxf ${tcpdump_file}
	fi
	
	if [ -d ${libpcap_dir} ]
	then
		echo ${libpcap_dir} directory already exist! Nothing to do.
	else
		echo untar ${libpcap_file}
		tar -zxf ${libpcap_file}
	fi
}

# build libpcap
{
	cd ${libpcap_dir}

	echo " _____________________"
	echo "|                     |"
	echo "| CONFIGURING LIBPCAP |"
	echo "|_____________________|"

	chmod +x configure
	./configure --host=$platform-linux --with-pcap=linux ac_cv_linux_vers=2

	if [ $? -ne 0 ]
	then
		exit_error
	fi	

	echo " __________________"
	echo "|                  |"
	echo "| BUILDING LIBPCAP |"
	echo "|__________________|"

	chmod +x runlex.sh
	make

	if [ $? -ne 0 ]
	then
		exit_error
	fi

	cd ..
}

# build tcpdump
{
	cd ${tcpdump_dir}
	
	echo " _____________________"
	echo "|                     |"
	echo "| CONFIGURING TCPDUMP |"
	echo "|_____________________|"
	
	chmod +x configure
	# Compile PIE (position independent executable) for Lollipop compatibility.
	./configure --host=$platform-linux ac_cv_linux_vers=2 --with-crypto=no CFLAGS='-fPIE' LDFLAGS='-fPIE -pie'

	if [ $? -ne 0 ]
	then
		exit_error
	fi	

	echo " __________________"
	echo "|                  |"
	echo "| BUILDING TCPDUMP |"
	echo "|__________________|"
	
	#setprotoent endprotoen not supported on android
	sed -i".bak" "s/setprotoent/\/\/setprotoent/g" print-isakmp.c
	sed -i".bak" "s/endprotoent/\/\/endprotoent/g" print-isakmp.c
	
	# NBBY is not defined => FORCE definition
	make CFLAGS='-DNBBY=8' # for tcpdump < 4.2.1 (CFLAGS redefined in Makefile) => just make
	
	if [ $? -ne 0 ]
	then
		exit_error
	fi
	
	cd ..
}

cp ${tcpdump_dir}/tcpdump .
chmod +x tcpdump

echo " __________________"
echo "|                  |"
echo "| TCPDUMP IS READY |"
echo "|__________________|"
echo `pwd`/tcpdump
```

编译开始

```
export NDK=/xxxxxx/android-ndk-r10e
bash build_tcpdump.sh
```

完成编译后即可看到
![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-36806be4848f03f3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后使用adb上传到手机里

```
adb push tcpdump /sdcard/
adb shell
su
cd /sdcard/
mv tcpdump /data/local/
cd /data/local/
chmod -R 777 tcpdump
```
不过在一款华为手机上执行的时候，发现一个问题

```
./tcpdump: not executable: magic 7F45
```

究其原因发现是混合编译的平台不一致导致的

我们可以通过命令查看CPU信息

```
adb shell cat /proc/cpuinfo
```
![这里写图片描述](http://upload-images.jianshu.io/upload_images/2006464-f2be9c9b94e7a83f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改上面的shell脚本

```
platform=aarch64
```
编译之后按照上述方式上传到手机里执行
```
./tcpdump -i any -p -s 0 -w /sdcard/temp/log.pcap
```

接着就可以使用
wireshark分析抓到的包了

混合编译参考
http://muzso.hu/2015/07/14/how-to-compile-tcpdump-for-android-5.-lollipop
