title: CMake 教程
categories: 日志
tags: [CMake]
date: 2021-10-25 13:39:00
---
## 使用系统环境变量

``` cmake
$ENV{PATH}
```

## 指定项目名称

相当于VS中的解决方案名称。

``` cmake
project(sqlitebrowser)
```

## 指定最低支持的CMake版本

``` cmake
cmake_minimum_required(VERSION 2.8.7)
```

## 设置变量

``` cmake
set(QSCINTILLA_DIR libs/qscintilla/Qt4Qt5)

set(SQLB_HDR
    src/version.h
    src/sqlitetypes.h
    src/csvparser.h
    src/sqlite.h
    src/grammar/sqlite3TokenTypes.hpp
    src/grammar/Sqlite3Lexer.hpp
    src/grammar/Sqlite3Parser.hpp
)
```

## 使用变量

``` cmake
${SQLB_HDR}
```

## 添加一个可执行程序项目

`${PROJECT_NAME}`指project指令中设置的名称。

``` cmake
add_executable(${PROJECT_NAME}
    ${SQLB_HDR}
    ${SQLB_SRC}
    ${SQLB_FORM_HDR}
    ${SQLB_MOC}
    ${SQLB_RESOURCES_RCC}
    ${SQLB_MISC}
)
```

## 添加预处理器定义

``` cmake
add_definitions(-DENABLE_SQLCIPHER)
add_definitions(-std=c++11)
```

## if语句

``` cmake
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()
```


<!--more-->


## 添加GUI工具选项

``` cmake
OPTION(ENABLE_TESTING "Enable the unit tests" OFF)
OPTION(FORCE_INTERNAL_ANTLR "Don't use the distribution's Antlr library even if there is one" OFF)
OPTION(FORCE_INTERNAL_QSCINTILLA "Don't use the distribution's QScintilla library even if there is one" OFF)
```

## 添加头文件包含目录

``` cmake
include_directories(
    "${CMAKE_CURRENT_BINARY_DIR}"
    ${QHEXEDIT_DIR}
    ${QCUSTOMPLOT_DIR}
    ${ADDITIONAL_INCLUDE_PATHS}
    src
)
```

## 添加链接库目录

``` cmake
link_directories("${CMAKE_CURRENT_BINARY_DIR}/${ANTLR_DIR}")
```

## 添加链接库

``` cmake
target_link_libraries(${PROJECT_NAME} antlr)
```

## 设置输出目录

``` cmake
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Lib)  # 静态库
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Lib)  # 动态库
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Bin)  # 可执行程序
```