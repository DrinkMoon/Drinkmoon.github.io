---
title: "CMake学习记录"
date: 2020-05-05
cover: https://res.cloudinary.com/drimoon-assets/image/upload/v1587660337/20200112164925_kcxb6u.jpg
---
# CMake
选择理由是它比其他make工具应用更广泛，安装也很简单，官方持续更新了很多现代化的语法。 但目前虽然有很多文档/材料，但缺乏优秀的从零开始的CMake中文教程，之前大学里写CMake就经常遇坑，只能用StackOverflow维持生活... 所以这篇博客是关于我学习CMake的记录，也可作为新手教程以供参考。

我倾向于用实际例子记录，对出现的问题进行解释、修正，对写起来不优雅的地方进行优化。

另外，因为存在个人翻译不准确的情况，附加了一些参考的超链接。

## Hello, World
CMake构建需要使用它特定的脚本语言，以HelloWorld作为开始：

- 新建项目文件夹，在其中创建CMakeLists.txt
- 指定本项目构建需要的CMake最低版本: cmake\_minimum\_required(VERSION xxx)
	- 尽量选择3.x最近的正式发布版本作为学习
- 指定项目名称/版本，使用的编程语言：project(HelloWorld VERSION 0.1.0 LANGUAGES CXX)
	- 还可以指定项目描述，项目主页等
- 单文件为例，以Main.cpp生成HelloWorld.exe
	- add\_executable(HelloWorld Main.cpp)
- 项目名称HelloWorld重复出现了两次，我们定义变量：SET(ProjectName HelloWorld)，通过${ProjectName}来替换重复的地方
- 接下来同CMake-GUI的正常使用，生成项目
- 除了HelloWorld，还会生成其他项目
	- ALL\_BUILD相当于makefile的默认目标，用于构建所有项目
	- ZERO\_CHECK用于检查CMakeLists.txt是否过期，保证其修改后能正确重新构建

CMakeLists.txt：
	
	# CMake vesion in my local machine is 3.17
    cmake_minimum_required(VERSION 3.17)
	# Language C++ : CXX
	SET(ProjectName HelloWorld)
	project(${ProjectName} VERSION 0.1.0 LANGUAGES CXX)
	add_executable(${ProjectName} Main.cpp)

Main.cpp：

	#include <iostream>
	
	int main()
	{
		std::cout << "Hello World!" << std::endl;
		return 0;
	}

## Hello, a bigger World
最近正好在学习DirectX12，试试用CMake来构建我的图形程序~

与构建相关的目录结构如下：

- FireSoul
	- build : 构建目录
	- bin : 输出目录
	- lib ：库目录
	- test : 测试用例
	- source ： 源码
		- core/math/...
			- include : 存放*.h
			- *.cpp
		- main
			- main.cpp
		- CMakeLists.txt
	- CMakeLists.txt

### 组织多重目录
现在我们的代码是分散在各个子目录里的，需要使用add\_subdirectory(目录)包含子目录，这些子目录下也需要创建CMakeLists.txt，会在add_subdirectory后执行。

./CMakeLists.txt:

	cmake_minimum_required(VERSION 3.17)
	SET(ProjectName FireSoul)
	project(${ProjectName} VERSION 0.1.0 LANGUAGES CXX)
	add_subdirectory(source)

### 包含多个头文件 & 添加多个源文件
现在我们需要包含头文件目录，使用include\_directories(目录)。
add\_executable现在要输入多个源文件名称，我们想要让CMake可以自动搜索目录下的源文件然后输入过来。
使用aux\_source\_directory(目录 变量名)：这个指令可以将目录下的所有源文件写入到变量中。

./source/CMakeLists.txt:

	include_directories(
	./core/include
	./math/include
	./main/include
	)
	aux_source_directory(. CxxFiles)
	aux_source_directory(./core/ CxxFiles)
	aux_source_directory(./math/ CxxFiles)
	aux_source_directory(./main/ CxxFiles)
	add_executable(${ProjectName} ${CxxFiles})

### 设置字符集
add\_definitions(-DUNICODE -D_UNICODE)

### 设置为窗口程序
set_target_properties(${ProjectName} PROPERTIES
	LINK_FLAGS /SUBSYSTEM:WINDOWS
)

### [优化] 使用target\_include\_directories
include\_directories在CMake新版本中不被推荐，可以使用进阶版的target\_include\_directories(项目 作用域 头文件目录)。

有三种[不同的作用域](https://stackoverflow.com/questions/26243169/cmake-target-include-directories-meaning-of-scope "不同的作用域")：

- PRIVATE
	- 头文件目录会填充指定项目的INCLUDE\_DIRECTORIES属性
	- 目录下的头文件可以出现在这个项目的头文件/源文件中
- INTERFACE
	- 头文件目录会填充指定项目的INTERFACE\_INCLUDE\_DIRECTORIES属性
	- 目录下的头文件可以出现在这个项目的头文件中，但不能出现在源文件中
	- target\_include\_directories(libname INTERFACE include PRIVATE include/libname)，可以直接在libname项目中包含include/libname下的头文件，但对于其他使用libname的项目只能添加libname/后缀
- PUBLIC
	- 填充PRIVATE + INTERFACE会填充的属性
	- 这些头文件有可能出现在项目中，也可能在使用该项目的其他项目中

### [问题] 包含目录识别为外部依赖
这里发现虽然项目构建/编译成功，但是头文件被项目识别为[外部依赖](https://stackoverflow.com/questions/13703647/how-to-properly-add-include-directories-with-cmake#](https://stackoverflow.com/questions/13703647/how-to-properly-add-include-directories-with-cmake# "外部依赖")，没有出现在项目中。 这里我们需要把所有*.h加到add\_executeable中。


./source/CMakeLists.txt:

	SET(HeaderFiles
	 ./common/include/Camera.h
	 ./common/include/d3dApp.h
	 ./common/include/d3dUtil.h
	 ./common/include/d3dx12.h
	 ./common/include/DDSTextureLoader.h
	 ./common/include/GameTimer.h
	 ./common/include/GeometryGenerator.h
	 ./common/include/MathHelper.h
	 ./common/include/UploadBuffer.h
	)
	
	aux_source_directory(. CxxFiles)
	aux_source_directory(./core/ CxxFiles)
	aux_source_directory(./common/ CxxFiles)
	aux_source_directory(./main/ CxxFiles)
	
	add_executable(${ProjectName} ${HeaderFiles} ${CxxFiles})
	set_target_properties(${ProjectName} PROPERTIES
		LINK_FLAGS /SUBSYSTEM:WINDOWS
	)
	target_include_directories(${ProjectName} PRIVATE
		./core/include
		./common/include
		./main/include
	)