```
本机OS: Ubuntu 14.04 x64
```
NDK开发模块的时候，如果涉及到网络请求，类似json数据传递的时候，有现成的第三方json库可以移植，后台C++开发中使用的比较多的是jsoncpp，今天记录一下jsoncpp移植到Android平台的过程

###cmake编译
此方法并非标准的NDK项目
采用的是cmake交叉编译生成
jsoncpp地址
https://github.com/open-source-parsers/jsoncpp
先将源码克隆下来
新建文件夹libjson
将jsoncpp源码中的include文件夹复制到该目录，然后进入src/lib_json目录复制
部分文件到libjson目录下的src目录中，如下
```
json_reader.cpp
json_tool.h
json_value.cpp
json_valueiterator.inl
json_writer.cpp
version.h.in
```
在libjson目录新建文件CMakeLists.txt
内容如下
```
CMAKE_MINIMUM_REQUIRED(VERSION 3.5)
PROJECT(Jpp)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")


#ARCHS集合
SET(ARCHS "arm" "arm64" "x86" "x86_64" "mips" "mips64")

#PLATFORM集合
SET(PLATFORM "arm-linux-androideabi" "aarch64-linux-android" "i686-linux-android" "x86_64-linux-android" "mipsel-linux-android" "mips64el-linux-android")


SET(JSONCPP_INCLUDE_DIR include)

INCLUDE_DIRECTORIES(${JSONCPP_INCLUDE_DIR})

SET(PUBLIC_HEADERS
    ${JSONCPP_INCLUDE_DIR}/json/config.h
    ${JSONCPP_INCLUDE_DIR}/json/forwards.h
    ${JSONCPP_INCLUDE_DIR}/json/features.h
    ${JSONCPP_INCLUDE_DIR}/json/value.h
    ${JSONCPP_INCLUDE_DIR}/json/reader.h
    ${JSONCPP_INCLUDE_DIR}/json/writer.h
    ${JSONCPP_INCLUDE_DIR}/json/assertions.h
    ${JSONCPP_INCLUDE_DIR}/json/version.h
    )

SOURCE_GROUP("Public API" FILES ${PUBLIC_HEADERS})

SET(jsoncpp_sources
                src/json_tool.h
                src/json_reader.cpp
                src/json_valueiterator.inl
                src/json_value.cpp
                src/json_writer.cpp
                src/version.h.in)


#INDEX=0 "arm" 
#INDEX=1 "arm64"
#INDEX=2 "x86"
#INDEX=3 "x86_64"
#INDEX=4 "mips"
#INDEX=5 "mips64"

SET(INDEX 0)

list(GET ARCHS ${INDEX} TARGET_ARCH)
list(GET PLATFORM ${INDEX} TARGET_PLATFORM)

MESSAGE("ARCH= " ${TARGET_ARCH})
MESSAGE("PLATFORM= " ${TARGET_PLATFORM})

SET(CMAKE_C_COMPILER "$ENV{HOME}/android-toolchain/${TARGET_ARCH}/bin/${TARGET_PLATFORM}-gcc")
SET(CMAKE_CXX_COMPILER "$ENV{HOME}/android-toolchain/${TARGET_ARCH}/bin/${TARGET_PLATFORM}-g++")


SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

ADD_LIBRARY(${TARGET_ARCH}_share SHARED ${PUBLIC_HEADERS} ${jsoncpp_sources})
SET_TARGET_PROPERTIES(${TARGET_ARCH}_share PROPERTIES OUTPUT_NAME ${TARGET_ARCH})
ADD_LIBRARY(${TARGET_ARCH}_static STATIC ${PUBLIC_HEADERS} ${jsoncpp_sources})
SET_TARGET_PROPERTIES(${TARGET_ARCH}_static PROPERTIES OUTPUT_NAME ${TARGET_ARCH})
```
编译之前确认已安装cmake
如果没有，请先安装
```
sudo apt-get install cmake
```
目录结构如下
![](http://upload-images.jianshu.io/upload_images/2006464-59033f3674a010ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

交叉编译请，参考笔者之前记录的[《NDK交叉编译之自定义工具链》](http://www.jianshu.com/p/3bbad4b1b099)

准备工作做好之后可以进行编译了
```
cmake .
make
```

![](http://upload-images.jianshu.io/upload_images/2006464-2dcf2689af51b20e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

笔者目前只编译了arm平台的，如果有需要其他平台的，只需要改动上述CMakeLists.txt中的编号即可
```
SET(INDEX 0)
```

###NDK原生编译


###测试
下面简单测试一下，就不建Android项目了
新建文件夹JNI
将刚才生成的libarm.a和include文件夹放入进去，新建文件Android.mk
```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := jsoncpp  
LOCAL_SRC_FILES := libarm.a 
include $(PREBUILT_STATIC_LIBRARY)  
  
include $(CLEAR_VARS)
LOCAL_MODULE    := main
LOCAL_C_INCLUDES  := $(LOCAL_PATH)/include/json
LOCAL_SRC_FILES := main.cpp
LOCAL_CPPFLAGS := -fexceptions

LOCAL_STATIC_LIBRARIES := jsoncpp  

LOCAL_CFLAGS += -pie -fPIE
LOCAL_LDFLAGS += -pie -fPIE

LOCAL_LDLIBS := -llog

include $(BUILD_EXECUTABLE)
```
新建Application.mk文件
```
APP_ABI := armeabi
APP_PLATFORM := 12
APP_STL := gnustl_static
APP_CPPFLAGS := -fexceptions -frtti
APP_CPPFLAGS += -std=gnu++11
APP_CPPFLAGS += -fpermissive
```
新建main.cpp
```

#include <jni.h>
#include <iostream>
#include "include/json/json.h"

using namespace std;

void JsonTest();

int main(){
    JsonTest();
    return 0;
}

void JsonTest(){
    Json::Value root;
    Json::Value array;
    Json::Value item;

    for (int i = 0; i < 2; i ++)
    {
        item["id"] = i;
        item["name"] = "name";
        array.append(item);
    }

    root["username"] = "andy";
    root["age"] = 18;
    root["array"] = array;

    string out = root.toStyledString();
    cout << out << endl;
}

```

![](http://upload-images.jianshu.io/upload_images/2006464-619bc2a2a2dad28f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后编译
```
ndk-build
```
然后将生成的main执行文件上传到手机中测试，手机需要root权限
笔者一般丢在/data/local目录下测试
```
./main
```

![](http://upload-images.jianshu.io/upload_images/2006464-991be008ae56ba77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
