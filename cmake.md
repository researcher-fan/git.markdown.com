# cmake user guide
原文链接：https://blog.csdn.net/afei__/article/details/81201039    

[toc]

***
## 简介
cmake 是一个跨平台、开源的构建系统。它是一个集软件构建、测试、打包于一身的软件。
它使用与平台和编译器独立的配置文件来对软件编译过程进行控制

***
## sample code

```
    # CMake 最低版本号要求
    cmake_minimum_required (VERSION 2.8)
    # 项目信息
    project (Demo)
    # 加入一个配置头文件，用于处理 CMake 对源码的设置
    configure_file (
        "${PROJECT_SOURCE_DIR}/config.h.in"
        "${PROJECT_BINARY_DIR}/config.h"
        )
    # 是否使用自己的 MathFunctions 库
    option (USE_MYMATH
            "Use provided math implementation" ON)
    # 是否加入 MathFunctions 库
    if (USE_MYMATH)
        include_directories ("${PROJECT_SOURCE_DIR}/math")
        add_subdirectory (math)
        set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
    endif (USE_MYMATH)
    # 查找当前目录下的所有源文件
    # 并将名称保存到 DIR_SRCS 变量
    aux_source_directory(. DIR_SRCS)
    # 指定生成目标
    add_executable(Demo ${DIR_SRCS})
    target_link_libraries (Demo ${EXTRA_LIBS})
```
* configure_file 命令用于加入一个配置头文件 config.h ，
    这个文件由 cmake 从 config.h.in 生成，通过这样的机制，将可以通过预定义一些参数和变量来控制代码的生成。
* option 命令添加了一个 USE_MYMATH 选项，并且默认值为 ON。
    根据 USE_MYMATH 变量的值来决定是否使用我们自己编写的 MathFunctions 库。

***
## 常用命令
### 基本命令
```
    # 指定cmake最小版本号
    cmake_minimum_required(VERSION 3.4.1)

    # 设置项目名称
    project(hello_world)

    # 设置编译类型
    add_executable(hello_world hello_world.cpp) # 生成可执行文件
    add_library(common STATIC util.cpp) # 生成静态库
    add_library(common SHARED util.cpp) # 生成动态库或共享库
    add_library(hello_world hello_world.cpp common.cpp util.cpp) # 指定lib包含哪些源文件

```

### 搜索文件
```
    # 搜索所有源文件
    aux_source_directory(. SRC_LIST) # 搜索当前目录下的所有.cpp文件
    add_library(hello_world ${SRC_LIST})

    # 自定义文件搜索规则
    file(GLOB SRC_LIST "*.cpp" "protocol/*.cpp")
    add_library(demo ${SRC_LIST})
    ## 或者
    file(GLOB SRC_LIST "*.cpp")
    file(GLOB SRC_PROTOCOL_LIST "protocol/*.cpp")
    add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
    ## 或者
    aux_source_directory(. SRC_LIST)
    aux_source_directory(protocol SRC_PROTOCOL_LIST)
    add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
```

### find相关命令
```
    # 查找指定库路径
    ## 类似命令包括 find_file()、find_path()、find_program()、find_package()
    find_library( 
              # Sets the name of the path variable.
              log-lib
              # Specifies the name of the NDK library
              log )

```

### 设置编译选项，及包括的头文件目录，link目录
```
    # 设置头文件包含目录
    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    )

    # 设置编译选项
    set(CMAKE_CXX_FLAGS "--stdc++")

    # 设置target 需要链接的库
    target_link_libraries( 
                       # 目标库
                       hello_world

                       # 目标库需要链接的库
                       # log-lib 是上面 find_library 指定的变量名
                       ${log-lib}
                       # 动态库
                       libtest.so
                       # 静态库
                       libtest.a
                       # 库名字
                       libtest
                       # 全路径库 ...
                       /usr/local/test/libtest.so)
```

### 变量设置及更新
```
    # 设置变量
    set(SRC_LIST main.cpp test.cpp)
    
    # 追加变量
    set(SRC_LIST ${SRC_LIST} test2.cpp)

    # list 追加或删除变量值
    set(SRC_LIST main.cpp)
    list(APPEND SRC_LIST test.cpp)
    list(REMOVE_ITEM SRC_LIST main.cpp)

```

### log 输出及include 其他头文件信息
```
    ## 打印信息
    message(${SRC_LIST})
    message("test log")
    message(WARNING "this is warnning message")
    message(FATAL_ERROR "this build has many error") # FATAL_ERROR 会导致编译失败

    ## 包含其他cmake 文件
    include(./common.cmake) # 指定包含文件的全路径
    include(def) # 在搜索路径中搜索def.cmake文件
    set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake) # 设置include的搜索路径
```

***
## 逻辑语法
### 条件控制语法
* if…elseif…else…endif
    * 逻辑判断和比较：
        * if (expression)：expression 不为空（0,N,NO,OFF,FALSE,NOTFOUND）时为真
        * if (not exp)：与上面相反
        * if (var1 AND var2)
        * if (var1 OR var2)
        * if (COMMAND cmd)：如果 cmd 确实是命令并可调用为真
        * if (EXISTS dir) if (EXISTS file)：如果目录或文件存在为真
        * if (file1 IS_NEWER_THAN file2)：当 file1 比 file2 新，
            或 file1/file2 中有一个不存在时为真，文件名需使用全路径
        * if (IS_DIRECTORY dir)：当 dir 是目录时为真
        * if (DEFINED var)：如果变量被定义为真
        * if (var MATCHES regex)：给定的变量或者字符串能够匹配正则表达式regex时
            为真，此处 var 可以用 var 名，也可以用 ${var}
        * if (string MATCHES regex)

    * 数字比较：
        * if (variable LESS number)：LESS 小于
        * if (string LESS number)
        * if (variable GREATER number)：GREATER 大于
        * if (string GREATER number)
        * if (variable EQUAL number)：EQUAL 等于
        * if (string EQUAL number)

    * 字母表顺序比较：
        * if (variable STRLESS string)
        * if (string STRLESS string)
        * if (variable STRGREATER string)
        * if (string STRGREATER string)
        * if (variable STREQUAL string)
        * if (string STREQUAL string)

* while...endwhile
* foreach…endforeach

```
    ## if else
        ### 示例
        if()
            # do 1
        else()
            # do 2
        endif()

    ## while...endwhile
        ### 示例
        while(condition)
        ...
        endwhile()

    ## foreach…endforeach
        ### 示例
        foreach(loop_var RANGE start stop [step])
        ...
        endforeach(loop_var)
```

***
## 常用变量

* 预定义变量
    * PROJECT_SOURCE_DIR：工程的根目录
    * PROJECT_BINARY_DIR：运行 cmake 命令的目录，通常是 ${PROJECT_SOURCE_DIR}/build
    * PROJECT_NAME：返回通过 project 命令定义的项目名称
    * CMAKE_CURRENT_SOURCE_DIR：当前处理的 CMakeLists.txt 所在的路径
    * CMAKE_CURRENT_BINARY_DIR：target 编译目录
    * CMAKE_CURRENT_LIST_DIR：CMakeLists.txt 的完整路径
    * CMAKE_CURRENT_LIST_LINE：当前所在的行
    * CMAKE_MODULE_PATH：定义自己的 cmake 模块所在的路径，
        SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)，
        然后可以用INCLUDE命令来调用自己的模块
    * EXECUTABLE_OUTPUT_PATH：重新定义目标二进制可执行文件的存放位置
    * LIBRARY_OUTPUT_PATH：重新定义目标链接库文件的存放位置

* 环境变量
    * $ENV{Name} ：使用环境变量
    * set(ENV{Name} value) ： 写入环境变量

* 系统信息
    * CMAKE_MAJOR_VERSION：cmake 主版本号，比如 3.4.1 中的 3
    * CMAKE_MINOR_VERSION：cmake 次版本号，比如 3.4.1 中的 4
    * CMAKE_PATCH_VERSION：cmake 补丁等级，比如 3.4.1 中的 1
    * CMAKE_SYSTEM：系统名称，比如 Linux-­2.6.22
    * CMAKE_SYSTEM_NAME：不包含版本的系统名，比如 Linux
    * CMAKE_SYSTEM_VERSION：系统版本，比如 2.6.22
    * CMAKE_SYSTEM_PROCESSOR：处理器名称，比如 i686
    * UNIX：在所有的类 UNIX 平台下该值为 TRUE，包括 OS X 和 cygwin
    * WIN32：在所有的 win32 平台下该值为 TRUE，包括 cygwin

* 主要开关选项
    * BUILD_SHARED_LIBS：这个开关用来控制默认的库编译方式，如果不进行设置，
        使用 add_library 又没有指定库类型的情况下，默认编译生成的库都是静态库。
        如果 set(BUILD_SHARED_LIBS ON) 后，默认生成的为动态库
    * CMAKE_C_FLAGS：设置 C 编译选项，也可以通过指令 add_definitions() 添加
    * CMAKE_CXX_FLAGS：设置 C++ 编译选项，也可以通过指令 add_definitions() 添加

    * 示例 ： add_definitions(-DENABLE_DEBUG -DABC) # 参数之间用空格分隔

***