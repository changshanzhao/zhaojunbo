---
title: Cmake
date: 2023-03-02 19:32:00
tags:
  - 教程
  - 学习笔记
categories:
	- 教程
---

<a href="https://juejin.cn/post/7184793007302901820" target="_blank" >参考文档</a>

<a href="https://cmake.org/cmake/help/latest/" target="_blank" >官方文档</a>

## 变量的简单使用：

```cmake
set(A ON)
if (A)
    message(STATUS "${A}")
    message(STATUS "${PROJECT_NAME}")
    message(STATUS "${PROJECT_SOURCE_DIR}")
    message(STATUS "${PROJECT_BINARY_DIR}")
```

## 强制使用静态链接：

```cmake
set(CMAKE_EXE_LINKER_FLAGS "-static")
```

## 改可执行文件名：

```cmake
add_executable(use_easyx main.cpp)
```

## 链接库：

```cmake
link_directories(${PROJECT_SOURCE_DIR}/lib)
target_link_libraries(use_easyx easyx)
```

## include链接目录（头文件）：

```cmake
include_directories(${PROJECT_SOURCE_DIR}/include)
```

## 添加子项目目录：

```cmake
add_subdirectory()
```

## 文件：

```cmake
flie(GLOB TEST
        "${PROJECT_SOURCE_DIR}/*.h" #只是当前目录下的.h文件存到变量TEST里，不会递归查找
        "${PROJECT_SOURCE_DIR}/*.cpp"
        )
aux_source_directory(${PROJECT_SOURCE_DIR}/src TEST2) #所有资源文件列表存到TEST2里面
foreach(c $(TEST))
    # 遍历
endforeach()
```

## 外部命令：

```cmake
execute_process(COMMAND git clone https://github.com/<username>/<repository>.git
                WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/deps/<repository>)
```

## 包管理：

```cmake
include(FetchContent)#引入功能模块

FetchContent_Declare(
        my-logger  		 #项目名称
        GIT_REPOSITORY https://github.com/ACking-you/my-logger.git #仓库地址
        GIT_TAG        v1.6.2  #仓库的版本tag
        GIT_SHALLOW TRUE    #是否只拉取最新的记录
)
FetchContent_MakeAvailable(my-logger)

add_excutable(main ${SRC})
#链接到程序进行使用
target_link_libraries(main my-logger)
```

这样引入第三方库的好处显而易见，优点类似于包管理的效果了，但缺少了最关键的中心仓库来确保资源的有效和稳定。参考golang再做个proxy层级就好了。 同样可以拉取最新的googletest可以使用下列语句：

```cmake
FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG        release-1.12.1
        GIT_SHALLOW TRUE
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

target_link_libraries(main gtest_main)
```

vcpkg有空了解一下（备忘）

未完待续.......
