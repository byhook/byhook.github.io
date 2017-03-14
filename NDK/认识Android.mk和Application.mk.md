NDK开发可以使用cmake进行交叉编译，或者使用原生的ndk-build进行编译
下面笔者从书中《Android C++高级编程》小结两个配置文件里一些常用的变量，方便后续查阅。
###1.Android.mk
Android.mk是一个NDK项目必备组件。
* 构建共享库
```
include $(BUILD_SHARED_LIBRARY)
```
* 构建多个共享库
```
LOCAL_PATH  := $(call my-dir)
#模块1
include $(CLEAR_VARS)
LOCAL_MODULE  := module1
LOCAL_SRC_FILES  := module1.c
include $(BUILD_SHARED_LIBRARY)
#模块2
include $(CLEAR_VARS)
LOCAL_MODULE  := module2
LOCAL_SRC_FILES  := module2.c
include $(BUILD_SHARED_LIBRARY)
```
* 构建静态库
通常都是将静态库并入共享库中
```
LOCAL_PATH  := $(call my-dir)
#第三方库
include $(CLEAR_VARS)
LOCAL_MODULE  := jsoncpp
LOCAL_SRC_FILES  := jsoncpp.cpp
include $(BUILD_STATIC_LIBRARY)
#原生模块
include $(CLEAR_VARS)
LOCAL_MODULE  := module
LOCAL_SRC_FILES  := module.c
LOCAL_STATIC_LIBRARIES  := jsoncpp
include $(BUILD_SHARED_LIBRARY)
```
* 共享库共享通用模块
```
LOCAL_PATH  := $(call my-dir)
#第三方库
include $(CLEAR_VARS)
LOCAL_MODULE  := jsoncpp
LOCAL_SRC_FILES  := jsoncpp.cpp
include $(BUILD_SHARED_LIBRARY)
#模块1
include $(CLEAR_VARS)
LOCAL_MODULE  := module1
LOCAL_SRC_FILES  := module1.c
LOCAL_SHARED_LIBRARIES  := jsoncpp
include $(BUILD_SHARED_LIBRARY)
#模块2
include $(CLEAR_VARS)
LOCAL_MODULE  := module2
LOCAL_SRC_FILES  := module2.c
LOCAL_SHARED_LIBRARIES  := jsoncpp
include $(BUILD_SHARED_LIBRARY)
```
* 多个NDK项目间共享模块
同时使用静态库和动态库时，可以在模块间共享通用代码，但这些都必须在同一个NDK项目中
将共享模块移动到NDK项目以外的其他目录
本身的Android.mk文件
```
LOCAL_PATH  := $(call my-dir)
#第三方库
include $(CLEAR_VARS)
LOCAL_MODULE  := jsoncpp
LOCAL_SRC_FILES  := jsoncpp.cpp
include $(BUILD_SHARED_LIBRARY)
```
为了避免构建系统冲突，应该将import-module函数宏放在Android.mk末尾
```
LOCAL_PATH  := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE  := module1
LOCAL_SRC_FILES  := module1.c
LOCAL_SHARED_LIBRARIES  := jsoncpp
include $(BUILD_SHARED_LIBRARY)
$(call import-module,code/jsoncpp)
```
import-module函数宏先定位共享模块，在将它导入到NDK项目中
* Prebuilt库
使用共享库的时候要求有共享库的源代码，Android NDK构建系统简单的把这些源文件包含在NDK项目中再次构建它们
预构建共享模块的Android.mk文件
```
LOCAL_PATH  := $(call my-dir)
#第三方库
include $(CLEAR_VARS)
LOCAL_MODULE  := jsoncpp
LOCAL_SRC_FILES  := libjsoncpp.so
include $(PREBUILT_SHARED_LIBRARY)
```
注意:Prebuilt库定义中不包含任何关于该库所构建的实际机器体系结构的信息，开发人员要确保Prebuilt库是为了与NDK项目相同的机器体系构建的
PREBUILT_SHARED_LIBRARY变量纸箱prebuilt-shared-library.mk Makefile片段。它什么都没有构建，但是它将Prebuilt库复制到了NDK项目的libs目录下。通过使用PREBUILT_STATIC_LIBRARY变量，静态库可以像共享库一样被用作Prebuilt库，NDK项目可以想普通共享库一样使用Prebuilt库了
```
LOCAL_SHARED_LIBRARIES  :=jsoncpp
```
* 构建独立的可执行文件
为了方便测试，Android NDK也支持构建独立的可执行文件
 ```
LOCAL_PATH  := $(call my-dir)
#模块
include $(CLEAR_VARS)
LOCAL_MODULE  := module
LOCAL_SRC_FILES  := module.c
include $(BUILD_EXECUTABLE)
```
BUILD_EXECUTABLE变量指向build-executable.mk Makefile片段。
* 其他构建系统变量
```
TARGET_ARCH：目标CPU体系结构的名称，例如arm
TARGET_PLATFORM：目标Android平台的名称，例如：android-3
TARGET_ARCH_ABI：目标CPU体系结构和ABI的名称，例如：armeabi-v7a
TARGET_ABI：目标平台和ABI的串联，例如：android-3-armeabi-v7a
LOCAL_MODULE_FILENAME：可选变量，用来重新定义生成的输出文件名称，默认情况下，构建系统使用LOCAL_MODULE的值来作为生成的输出文件名称，使用此变量可以覆盖
LOCAL_CPP_EXTENSION：C++源文件的扩展名为.cpp，这个变量可以用来为C++源代码指定一个或多个扩展名
LOCAL_CPP_EXTENSION  := .cpp cxx
LOCAL_CPP_FEATURES：可选变量，用来指明模块所依赖的具体C++特性，如RTTI，exceptions等
LOCAL_CPP_FEATURES  := rtti
LOCAL_C_INCLUDES：可选目录列表，NDK安装目录的相对路径，用来搜索头文件
LOCAL_C_INCLUDES  := sources/shared-module
LOCAL_C_INCLUDES  := $(LOCAL_PATH)/include
LOCAL_CFLAGS：一组可选的编译器标志，在编译C和C++源文件的时候会被传送给编译器
LOCAL_CFLAGS  := -DNDEBUG -DPORT=1234
LOCAL_CPP_FLAGS：一组可选的编译器标志，在编译C++源文件的时候会被传送给编译器
LOCAL_WHOLE_STATIC_LIBRARIES：LOCAL_STATIC_LIBRARIES的变体，用来指明应该被包含在生成的共享库中所有静态库内容，几个静态库有循环依赖时，很有用
LOCAL_LDLIBS：链接标志的可选列表，当对目标文件进行链接以生成输出文件时该标志将被传送给链接器。它主要用于传送要进行动态链接的系统库列表，例如要与Android NDK日志库链接，使用以下代码
LOCAL_LDFLAGS  := -llog
LOCAL_ALLOW_UNDEFINED_SYMBOLS：可选参数，它禁止在生成的文件中进行缺失符号检查，若没有定义，链接器会在符号缺失生成错误信息
LOCAL_ARM_MODE：可选参数，ARM机器体系结构特有变量，用于指定要生成的ARM二进制类型
LOCAL_ARM_MODE  := arm
LOCAL_SRC_FILES  := file.c file2.c.arm
LOCAL_ARM_NEON：可选参数，ARM机器体系特有变量，用来指定在源文件中使用的ARM高级单指令流内联函数
LOCAL_ARM_NEON  := true
LOCAL_SRC_FILES  := file.c file2.c.neon
LOCAL_DIABLE_NO_EXECUTE：可选变量，用来禁用NX Bit安全特性，用来隔离代码区和存储区，防止恶意软件通过将它的代码注入到应用程序的存储区来控制应用程序
LOCAL_DISABLE_NO_EXECUTE  := true
LOCAL_EXPORT_CFLAGS：该变量记录一组编译器标志，这些编译器标志会被添加到通过变量LOCAL_STATIC_LIBRARIES或LOCAL_SHARED_LIBRARIES使用模块的其他模块LOCAL_CFLAGS定义中
LOCAL_MODULE  := jsoncpp
...
LOCAL_EXPORT_CFLAGS  := -DENABLE_AUDIO
...
LOCAL_MODULE  := module
LOCAL_CFLAGS  := -DDEBUG
...
LOCAL_SHARED_LIBRARIES  := jsoncpp
编译器在构建module的时候会以-DENABLE_AUDIO -DDEBUG标志执行
LOCAL_EXPORT_CPPFLAGS：和LOCAL_EXPORT_CFLAGS一样，不过它是C++特定代码编译器标志
LOCAL_EXPORT_LDFLAGS：和LOCAL_EXPORT_CFLAGS一样，不过用作链接器标志
LOCAL_EXPORT_C_INCLUDES：该变量运行记录路径集，这些路径会被添加到通过变量LOCAL_STATIC_LIBRARIES或LOCAL_SHARED_LIBRARIES使用该模块的LOCAL_C_INCLUDES定义中
LOCAL_SHORT_COMMANDS：对于有大量的资源或独立的静态/共享库的模块，该变量应该被设置为true
LOCAL_FILTER_ASM：该变量定义了用于过滤来自LOCAL_SRC_FILES变量的装配文件的应用程序
```
* 其他的构建系统函数宏
```
all-subdir-makefiles：返回当前目录的所有子目录下的Android.mk构建文件列表，例如，调用一下命令可以将子目录下的所有Android.mk文件包含在构建过程中:
include $(call all-subdir-makefiles)
this-makefile：返回当前Android.mk构建文件的路径
parent-makefile：返回包含当前构建文件的父Android.mk构建文件的路径
grand-parent-makefile：和parent-makefile一样但用于祖父目录
```
* 定义新变量
开发人员可以定义其他变量来简化构建文件，以LOCAL_和NDK_开头的名称预留给Android NDK构建系统使用。
```
...
MY_SRC_FILES  ;= jsoncpp.cpp
LOCAL_SRC_FILES  := $(addprefix jsoncpp/,$(MY_SRC_FILES))
...
```
* 条件操作
Android.mk构建文件也可以包含关于这些变量的条件操作
```
...
ifeq ($(TARGET_ARCH),arm)
      LOCAL_SRC_FILES  += abc.c
else
      LOCAL_SRC_FILES  += xyz.c
endif
...
```

###2.Application.mk
它是Android NDK构建系统使用的一个可选的构建文件，也是一个GNU Makefile片段
下面是Application.mk构建文件支持的变量：
```
APP_MODULES：默认情况下，Android NDK构建系统构建Android.mk文件声明的所有模块。
该变量可以覆盖上述行为并提供一个空格分开的，需要被构建的模块列表
```

```
APP_OPTIM：该变量可以设置为release和debug来改变生成的二进制文件的优化级别，
默认为release模式。
```

```
APP_CLAGS：列出编译器标识，在编译任何模块的C和C++源文件时这些标志都会被传给编译器
```

```
APP_CPPFLAGS：列出编译器标识，在编译任何模块的C++源文件时这些标志都会被传给编译器
```

```
APP_BUILD_SCRIPT：默认情况下，Android NDK构建系统在项目的JNI子目录下查找Android.mk构建文件。
可以用该变量改变上述行为，并使用不同的生成文件
```

```
APP_ABI：默认情况下，Android NDK构建系统为armeabi ABI生成二进制文件，
可以用该变量改变上述行为，并为其他ABI生成二进制文件：例如
APP_ABI  := mips
另外可以设置多个ABI
APP_ABI  := armeabi mips
为所有支持的ABI生成二进制文件
APP_ABI  := all
```
```
APP_STL：默认情况下，Android NDK构建系统使用最小
STL运行库，也被成为system库。可以用该变量选择不同的
STL实现。
APP_STL  := stlport_shared
```
```
APP_GNUSTL_FORCE_CPP_FEATURES：与LOCAL_CPP_EXTENSIONS变量类似，改变量表明所有模
块都依赖于具体的C++特性，如RTTI，exceptions等。
APP_SHORT_COMMANDS：与LOCAL_SHORT_COMMANDS变量相似，该变量使得构建
系统存在有大量源文件的情况下可以在项目里使用更短的命令。
```
