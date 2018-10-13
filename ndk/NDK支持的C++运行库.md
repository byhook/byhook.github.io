Android平台自身带有一个微型的C++运行库支持库，称为系统运行库，但是功能有所限制，还有一些补充系统库的额外C++运行库

|C++运行库|C++异常支持|C++ RTTI支持|C++标准库|
|:----:|:----:|:----:|:----:|
|系统库|No|No|No|
|GAbi++|No|Yes|No|
|STLport++|No|Yes|Yes|
|GNU STL|Yes|Yes|Yes|

 ###1.GAbi++ C++运行库
GAbi++ C++运行库是一个试验性的，最简化的运行库，它提供建立在系统运行库所提供的相同特性集基础之上的RTTI支持。它可以作为静态库或共享库使用
###2.STLport C++运行库
STLport是一个开源的，多平台的C++标准实现。它提供一个C++标准库头文件的完整集合以及对RTTI的支持。它也可以作为静态库或共享库使用。
###3.GNU STL C++运行库
GNU标准 C++支持库，也叫libstdc++-v3，是Android NDK中最全面的标准C++运行库。
在GNU标准C++运行库中，C++异常与C++RTTI均被支持。

###指定C++运行库
Android NDK构建系统变量APP_STL可被指定需要使用的C++运行库。
```
APP_ABI    :=  armeabi armeabi-v7a
APP_STL   :=  system
```
```
system:   默认的微型系统C++运行库
gabi++_static:   作为静态库的Gabi++运行库
gabi++_shared:   作为共享库的Gabi++运行库
stlport_static:   作为静态库的STLport运行库
stlport_shared:   作为共享库的STLport运行库
gnustl_static:   作为静态库的GNU STL运行库
gnustl_shared:   作为共享库的STLport运行库
```
注意:
当C++运行库以共享库的形式使用时，应用程序需要先家长所需要的共享库，然后在加载依赖此共享库的其他原生模块。
```
static{
     System.loadLibrary("stlport_shared");
     System.loadLibrary("xxxxx");
}
```
###C++异常支持
Java中异常处理很方便，考虑到性能和兼容性，默认情况下C++ Exception支持是不可用的，NDK中需要添加对C++异常的支持。
Android.mk配置
```
LOCAL_MODULE    :=   module
...
LOCAL_CPP_FEATURES   +=  exceptions
...
include $(BUILD_SHARED_LIBRARY)
```
Application.mk配置
```
APP_STL    :=  gnustl_shared
APP_CPPFLAGS   +=   -fexceptions
```
也可用同样的方式启用C++RTTI的支持。
###C++RTTI支持
RTTI机制即在运行库展示对象类型信息。该机制主要执行安全类型转化。
Android.mk配置
```
LOCAL_MODULE    :=   module
...
LOCAL_CPP_FEATURES   +=  rtti
...
include $(BUILD_SHARED_LIBRARY)
```
Application.mk配置
```
APP_STL    :=  gnustl_shared
APP_CPPFLAGS   +=   -frtti
```

本文引自《Android C++高级编程》
