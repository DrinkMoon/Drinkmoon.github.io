---
title: "CMake学习记录"
date: 2020-05-05
cover: https://res.cloudinary.com/drimoon-assets/image/upload/v1587660337/20200112164925_kcxb6u.jpg
---
# CMake
选择CMake的理由是它相比其他构建项目的工具应用更广泛，安装基本不需要依赖，也支持了一些现代化的语法。但我发现虽然有很多关于不同语法的文档，但缺乏比较好的从零开始的CMake中文教程，预期我会踩进很多坑，所以写这篇博客作为个人学习记录，同时方便他人参考。

## Hello World
CMake构建使用自己开发的脚本语言，以HelloWorld作为开始：

- 新建项目文件夹，在其中创建CMakeLists.txt
- 指定本项目构建需要的最低CMake版本: cmake\_minimum\_required(VERSION xxx)
	- 我们尽量选择3.x最近的正式发布版本作为学习
- 指定项目名称/版本，使用的编程语言：project(HelloWorld VERSION 0.1.0 LANGUAGES CXX)
	- 还可以指定项目描述，项目主页，这里略过
- 简单添加一个main.cpp并指定生成的exe名称
	- add\_executable(HelloWorld Main.cpp)
- 打开CMake-GUI，source code目录指向项目路径，build目录可以在项目目录下新建文件夹Build并指向它
- 点击Config，选择IDE和Platform，确定(本文以VS2019, x64的Windows环境举例)
- 点击Generate，成功后会发现Build文件夹出现了项目名.sln，打开
- 会出现ALL\_BUILD(相当于makefile的默认目标，用于构建所有项目)，HelloWorld，ZERO\_CHECK(CMake用于检查CMakeLists.txt是否过期，用于保证CMakeLists.txt修改后能够正确重新构建)三个项目

CMakeLists.txt：
	
	# CMake vesion in my local machine is 3.17
    cmake_minimum_required(VERSION 3.17)
	# Language C++ : CXX
    project(HelloWorld VERSION 0.1.0 LANGUAGES CXX)
    add_executable(HelloWorld Main.cpp)

Main.cpp：

	#include <iostream>
	
	int main()
	{
		std::cout << "Hello World!" << std::endl;
		return 0;
	}

## 初级

### 设置/使用变量
我们发现项目名称HelloWorld之前重复出现了，现在定义一个变量作为项目名称：SET(ProjectName HelloWorld)，可以通过${ProjectName}来使用它

	cmake_minimum_required(VERSION 3.17)
	SET(ProjectName HelloWorld)
	project(${ProjectName} VERSION 0.1.0 LANGUAGES CXX)
	add_executable(${ProjectName} Main.cpp)

### 多级目录管理
最近打算写一个命名为Hammer的软渲染器，目录结构如下：

- Hammer
	- src
		- core
			- include
				- *.h
			- *.cpp
		- utility
			- include
				- *.h
			- *.cpp
		- Main.cpp

步骤：

- 在项目根目录，src目录下分别创建CMakeLists.txt
- Hammer/CMakeLists.txt:
	- 包含子目录：add\_subdirectory(src)，这句指令会执行src/CMakeLists.txt的内容
- Hammer/src/CMakeLists.txt
	- 包含头文件: include\_directories(./core/include ./utility/include)
	- 搜索所有源文件：aux\_source\_directory(. ALL\_SRCS), aux_source_directory(./core/ ALL\_SRCS), aux\_source\_directory(./utility/ ALL\_SRCS)
	- 包含源文件：add\_executable(${ProjectName} ${ALL\_SRCS})

这里的使用aux\_source\_directory命令是因为add\_executable一个个写文件名过于麻烦，这个命令可以把所有要加的文件名写到某个变量里，我这里命名为ALL\_SRCS。

Hammer/CMakeLists.txt:

    cmake_minimum_required(VERSION 3.17)
    SET(ProjectName Hammer)
    project(${ProjectName} VERSION 0.1.0 LANGUAGES CXX)
    add_subdirectory(src)

Hammer/src/CMakeLists.txt:

	include_directories(
	./core/include
	./utility/include
	)
	aux_source_directory(. ALL_SRCS)
	aux_source_directory(./core/ ALL_SRCS)
	aux_source_directory(./utility/ ALL_SRCS)
	add_executable(${ProjectName} ${ALL_SRCS})