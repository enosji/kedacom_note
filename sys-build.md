# 配置sys-build

## sys-build由来

sys-build编译系统是在Android ndk的编译脚本基础上裁剪出来的，它有点类似于之前用的九鼎厂家出的BSP由一个makefile将内核、uboot、rootfs联合起来编译的一个工具

## sys-build源码树

sys-build树状目录结构图

```c
makerulers
|--build
    |--config
    |--core
    |--gmsl
    \--tools
|--docs
|--README
|--samples
|--sys-build    
```

（1）build	该文件夹，包含4个子目录，保存编译系统的核心脚本文件和编译系统当前支持的平台目标板配置

（2）docs	该文件夹内存放的是sys-build相关说明文档

（3）samples	该文件夹下面，存放的是实例代码

（4）sys-build	可执行脚本文件，执行它可实现组件的编译

## sys-build的安装及前期配置

### 1、执行脚本文件

原先我们需要在基础库中现在sys-build源码树

```c
$ git clone git@172.16.8.9:/sysdev/makerulers.git -b sysdev_kdm_v1.0
```

但是现在不需要了，因为你在数据库中下载项目的时候，基本上都会带上sys-build的源码，只需要在编译前，执行下sys-build内的init.sh脚本文件就好了

```c
$ source makerulers/build/tools/init.sh
```

### 2、config添加config文件

在执行完sys-build的init.sh脚本之后，我们需要根据开发平台进行config的相关配置，这里我们需要编译的是hdu6的平台开发，使用的芯片是rk3568，移动到相关平台的配置文件存放路径中

```c
$ cd ~/hdu6/makerulers/build/configs/arm64/rk356x
$ ls
common_config  hdu6_rk3568_ep_config  hdu6_rk3568_rc_config  main.mk  nvr_rk3568_config  rk3568_demo_config
```

发现相关配置文件有两个分别是hdu6_rk3568_ep_config和hdu6_rk3568_rc_config，原因是因为hdu6这个板子使用的是两个芯片协同工作需要配置一个主内核一个从内核，这里不进行深入解释，后期会进行深入理解，这里我们配置hdu6_rk3568_rc_config

```c
$ sys-build config hdu6_rk3568_rc_config
```

出现以下信息，表示配置成功

```c
$ sys-build config hdu6_rk3568_rc_config 
Configure sys-build ...

========================================
APP_BUILD_SCRIPT := $(APP_PROJECT_PATH)/make.mk
APP_OPTIM := release
APP_OUTPUT_DIR := out
APP_DEBUG_MODULES :=
APP_MODULES :=
APP_PLATFORM := rk356x
APP_ARCH := arm64
APP_ABI := arm64-v8a
APP_BOARD := hdu6_rk3568_rc
APP_VENDOR := rockchip
APP_WORKSPACE := $(APP_PROJECT_PATH)
APP_SDK_DIR := $(APP_PROJECT_PATH)
APP_LSP_DIR := $(APP_WORKSPACE)/packages/linux_lsp
APP_RELEASE_DIR := $(APP_WORKSPACE)/release
APP_STL :=
APP_ALIAS_BOARD := hdu6_rk3568_rc
APP_TOOLCHAIN_SYSROOT :=
APP_ARM64_TOOLCHAIN := /opt/rockchip/rk356x/bin/aarch64-buildroot-linux-gnu-

========================================
```

这时，当前路径下多出一个Application.mk配置文件

Application.mk文件作为编译时编译环境的配置文件被编译系统读取，它可以通过命令sys-build config命令自动生成，也可以手动创建，缺省情况下的Application.mk文件为编译系统源码树的build/core/default-application.mk

## 分析Application.mk文件

当我们config配置成功后会在当前目录下生成一个Application.mk的配置文件

```makefile
#--------------------------------------#
#        Sys-build Version: 2.2        #
#--------------------------------------#


# Point to a script file.the APP_PROJECT_PATH to record current path.
APP_BUILD_SCRIPT := $(APP_PROJECT_PATH)/make.mk
APP_OPTIM := release

# define output directory ,the defalut as follow: 
# /home/jiwansu/hdu6/out.
APP_OUTPUT_DIR := out

#APP_DEBUG_MODULES := 1
APP_DEBUG_MODULES :=
APP_MODULES :=
APP_PLATFORM := rk356x
APP_ARCH := arm64

# defined current ABI(Application Binary Interface),example 
# armeabi(embedded-application binary interface) or armeabi-v7a.
APP_ABI := arm64-v8a
APP_BOARD := hdu6_rk3568_rc
APP_VENDOR := rockchip

# 这个变量总是被描述成一个完整的sysdev工程的顶层目录,即，如果目 
# 前只是在某个模块里面进行编译，也应该根据其相对sysdev的位置而计 
# 算出sysdev的顶层目录，并赋值给这个变量. 如果你就在完整的sysdev 
# 工程的顶层目录下进行编译，这个值默认就是当前目录，不需要再另外 
# 赋值. 
# 这个值之所以需要这么确定，是因为工程的sdk和linux_lsp的路径是通 
# 过该值计算出来的,如果该变量给的不正确，sys-build会给出警告信息， 
# 如果你确定当前模块不需要使用SDK/LINUX_LSP等路径，可以忽略这些警 
# 告.
APP_WORKSPACE := $(APP_PROJECT_PATH)

# SDK相对sysdev的路径，每个平台有所差别，统一提供给需要的模块使用模 
# 块本身不应该覆盖定义该变量，以免造成其他模块无法正常引用该变量.
APP_SDK_DIR := $(APP_PROJECT_PATH)

# Linux_lsp相对sysdev的路径，统一提供给需要的模块使用模块本身不应该 
# 覆盖定义该变量，以免造成其他模块无法正常引用该变量.
APP_LSP_DIR := $(APP_WORKSPACE)/packages/linux_lsp

# 发布路径的顶层目录，默认就存放在APP_WORKSPACE之下各个模块需要根据 
#《版本发布说明》当中的规则，自行构建自身的发布结构，比如 
# 25381(APP_RELEASE_DIR)/cbb/sysdbg/include
APP_RELEASE_DIR := $(APP_WORKSPACE)/release

# Reserved function.
APP_STL :=

# Alias of the target board.
APP_ALIAS_BOARD := hdu6_rk3568_rc

# Specifyed toolchain sysroot dirctory, 
# it's equivalent to '--sysroot' option.
APP_TOOLCHAIN_SYSROOT :=
APP_ARM64_TOOLCHAIN := /opt/rockchip/rk356x/bin/aarch64-buildroot-linux-gnu-

```

其中的一些变量的定义和uboot、kernel进行config生成的东西较为类似，都是用来描述到时候项目所编译的环境。

### Applicaton.mk变量说明

|        变量名         |                说明                 |
| :-------------------: | :---------------------------------: |
| ==APP_PROJECT_PATH==  |          记录当前工作路径           |
|   APP_BUILD_SCRIPT    |           指定编译的脚本            |
|       APP_OPTIM       |           指定编译的版本            |
|    APP_OUTPUT_DIR     |       指定目标文件输出的目录        |
|   APP_DEBUG_MODULES   |    控制编译时模块的详细信息输出     |
|      APP_MODULES      |       指定需要编译的所有模块        |
|     APP_PLATFORM      |           当前模块的平台            |
|       APP_ARCH        |           当前模块的架构            |
|        APP_ABI        |            当前模块的ABI            |
|       APP_BOARD       |           当前模块的板子            |
|      APP_VENDOR       |             板子生产商              |
| APP_ <ARCH>_TOOLCHAIN |   定义相关平台的交叉编译工具前缀    |
|        APP_STL        | 指定要引用的sys-build支持的内部模块 |
|   ==APP_WORKSPACE==   |         指定工程的有效路径          |
|    ==APP_SDK_DIR==    |          指向有效的sdk路径          |
|    ==APP_LSP_DIR==    |          指向有效的lsp路径          |
|  ==APP_RELEASE_DIR==  |   定义各个平台约定的发布路径前缀    |



## demo编译

1、移动目录到sys-build自带的实列代码中

```c
$ cd makerulers/samples/hello-world
```

2、对这个demo进行配置，当前实例中config使用rk3568板子的配置，在当前目录下生成Application.mk文件

```c
$ sys-build config hdu6_rk3568_rc_config
```

3、编译

```c
$ sys-build
```

4、发现报错，错误如下

```c
SYS-BUILD: WARNING: May be you are try Compiling a sub-make.    
SYS-BUILD:      But the TARGET_SDK_DIR or TARGET_LSP_DIR variables no define.    
SYS-BUILD:      They depends APP_WORKSPACE  directory. If you need to use them.    
SYS-BUILD:      Please define them or redefine the APP_WORKSPACE variable to point a valid workspace !    
Install        : testc => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc
Install        : testc++ => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc++
Install        : testc_exe => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc_exe
```

分析：报错提示，我们可能在编译一个子版本，需要我们指定APP_WORKSPACE这个变量，打开Application.mk文件发现APP_WORKSPACE依赖$(APP_PROJECT_PATH)这个变量，而APP_PROJECT_PATH这个变量记录的是当前工作路径。

但是我们APP_WORKSPACE这个变量总是被描述成一个完整的sysdev工程的顶层目录,即，如果目前只是在某个模块里面进行编译，也应该根据其相对sysdev的位置而计算出sysdev的顶层目录，并赋值给这个变量. 如果你就在完整的sysdev 工程的顶层目录下进行编译，这个值默认就是当前目录，不需要再另外赋值. 这个值之所以需要这么确定，是因为工程的sdk和linux_lsp的路径是通过该值计算出来的,如果该变量给的不正确，sys-build会给出警告信息，如果你确定当前模块不需要使用SDK/LINUX_LSP等路径，可以忽略这些警告。

所以我们需要指定APP_PROJECT_PATH的路径为项目的顶层目录，需要在Application.mk中添加一句话

```makefile
# 这个变量总是被描述成一个完整的sysdev工程的顶层目录,即，如果目 
# 前只是在某个模块里面进行编译，也应该根据其相对sysdev的位置而计 
# 算出sysdev的顶层目录，并赋值给这个变量. 如果你就在完整的sysdev 
# 工程的顶层目录下进行编译，这个值默认就是当前目录，不需要再另外 
# 赋值. 
# 这个值之所以需要这么确定，是因为工程的sdk和linux_lsp的路径是通 
# 过该值计算出来的,如果该变量给的不正确，sys-build会给出警告信息， 
# 如果你确定当前模块不需要使用SDK/LINUX_LSP等路径，可以忽略这些警 
# 告.
APP_PROJECT_PATH := ../../../			#在这里指定变量的路径为项目的顶层路径
APP_WORKSPACE := $(APP_PROJECT_PATH)

# SDK相对sysdev的路径，每个平台有所差别，统一提供给需要的模块使用模 
# 块本身不应该覆盖定义该变量，以免造成其他模块无法正常引用该变量.
APP_SDK_DIR := $(APP_PROJECT_PATH)

# Linux_lsp相对sysdev的路径，统一提供给需要的模块使用模块本身不应该 
# 覆盖定义该变量，以免造成其他模块无法正常引用该变量.
APP_LSP_DIR := $(APP_WORKSPACE)/packages/linux_lsp

# 发布路径的顶层目录，默认就存放在APP_WORKSPACE之下各个模块需要根据 
#《版本发布说明》当中的规则，自行构建自身的发布结构，比如 
# 10440(APP_RELEASE_DIR)/cbb/sysdbg/include
APP_RELEASE_DIR := $(APP_WORKSPACE)/release
```

5、再次编译

```c
$ sys-build
Compile   : testc <= testc.c
TestProgram    : testc
Install        : testc => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc
Compile++   : testc++ <= testc++.cpp
TestProgram    : testc++
Install        : testc++ => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc++
Compile   : testc_exe <= testc.c
Executable     : testc_exe
Install        : testc_exe => out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/testc_exe
```

编译成功，由信息可知编译生成的文件被放到了out/libs/arm64/arm64-v8a/rk356x/hdu6_rk3568_rc/路径下



## make.mk编译脚本说明

make.mk编译脚本和Android.mk编译脚本一样，都使用统一的格式编写，其语法和makefile一样，换句话说，它就是一个makefile文件。他的工作逻辑和Makefile类似，都是通过顶层的make.mk管理各个子目录中的make.mk来实现工程文件的管理，他是独立存在的，譬如你写了个hello的测试代码你可以用makefile去管理这个代码，也可以用make.mk去管理，当一个项目里出现make.mk和makefile时说明我们使用make.mk去控制makefile进行工作的

### make.mk变量引用说明

| 变量名       | 说明                                                  |
| ------------ | ----------------------------------------------------- |
| LOCAL_PATH   | 必须的，只通过$(call my-dir)获取，表示当前make.mk路径 |
| LOCAL_MODULE | 必须的，定义一个模块或者组件的名字                    |
|LOCAL_MODULE_FILENAME | 	定义最终文件生成的名字		|
|LOCAL_SRC_FILES	|记录所有需要编译的源文件	|
|LOCAL_CPP_EXTENSION	|定义扩展的c++源文件后缀支持 如.cxx .cpp .CC|
|LOCAL_CPP_FEATURES|	定义当前c++支持的特性，如 rtti  exceptions|
|LOCAL_C_INCLUDES	|记录编译源文件所依赖的头文件路径|
|LOCAL_ALLOW_UNDEFINED_SYMBOLS	|如果该变量值为true，则关闭编译动态库时检查符号未定义功能。默认为false|
|LOCAL_DISABLE_FORMAT_STRING_CHECKS	|如果这个变量为true，则关闭对printf函数的格式串检查，默认开启|
|LOCAL_CFLAGS	|编译c/c++文件指定的的标志|
|LOCAL_CPPFLAGS	|同上，只支持c++文件|
|LOCAL_STATIC_LIBRARIES	|定义编译所依赖的静态库|
|LOCAL_SHARED_LIBRARIES	|定义编译所依赖的动态库|
|LOCAL_WHOLE_STATIC_LIBRARIES	|用于编译动态库时指定依赖的静态库，它将依赖的静态库所有符号连入|
|LOCAL_LDLIBS	|定义链接时候所依赖的库|
|LOCAL_LDFLAGS	|定义链接时候的标志|
|LOCAL_EXPORT_CFLAGS	|定义需要export出去的标志，依赖该模块的模块会自动加上该标志|
|LOCAL_EXPORT_CPPFLAGS	|同上，不过用于cppflags|
|LOCAL_EXPORT_C_INCLUDES	|同上，不过用于c_includes|
|LOCAL_EXPORT_LDLIBS	|同上，不过用于ldlibs|
|LOCAL_FILTER_ASM	|通过shell命令过滤生成的汇编文件|
|LOCAL_LINK_MODE	|声明该模块采用C方式还是C++方式进行链接。仅在编译动态库或者可执行程序模式下有效。缺省为C++连接方式|
|LOCAL_RELATE_MODE	|声明该模块的涉及范围，取值为board plat arch compiler，缺省模式为board|
|**LOCAL_TARGET_TOP** 	|指定源码目录树，仅用于编译内核和boot|
|**LOCAL_TARGET_CONFIG** 	|	指定makefile配置文件，仅用于编译内核和boot|
|**LOCAL_TARGET_RULER** 	|	指定makefile编译目标，仅用于编译内核和boot|
|**LOCAL_TARGET_TOOLCHAIN**	|	指定私有make的交叉编译器，仅用于编译内核和boot|
|**LOCAL_TARGET_COPY_FILES** 	| 指定需要copy的文件，在内核、boot和第三方二进制模块下定义的文件将被复制到out/obj下的板级目录。所有模块中由它定义的文件将被自动记录到版本发布时需要复制的文件列表中。 |
|**LOCAL_TARGET_CMD** 	|	指定需要执行的shell命令，仅用于编译第三方二进制|
|**LOCAL_DEPS_MODULES**	|	指定编译内核、bootloader或者是第三方二进制需要依赖的模块|
|LOCAL_RELEASE_PATH	|	指定编译目标最终的发布路径，多个路径用“;”隔开，如果是内核或boot或者第三方二进制模块，由LOCAL_TARGET_COPY_FILES指定的所有文件将被发布。|
|CLEAR_VARS|指向一个清除指定变量的脚本，用法include $(变量)|
|BUILD_SHARED_LIBRARY	|	指向一个编译共享库的脚本，用法同上|
|BUILD_STATIC_LIBRARY	|	同上，不过是编译静态库|
|BUILD_KERNEL	|	同上，不过是编译内核|
|BUILD_BOOTLOADER	|	同上，不过是编译bootloader|
|BUILD_TH3_BINARY	|	同上，不过是编译第三方二进制，例如sdk|
|BUILD_TEST	|	同上，不过是编译demo程序|
|BUILD_EXECUTABLE	|	同上，不过是编译可执行程序|
|PREBUILT_SHARED_LIBRARY	|	同上，不过是预处理动态库，用于引用第三方动态库|
|PREBUILT_STATIC_LIBRARY	|	同上，不过是预处理静态库，用于引用第三方静态库|
|TARGET_ARCH	|	只读的，表示当前编译环境的架构|
|TARGET_PLATFORM	|	只读的，表示当前编译环境的平台|
|TARGET_ABI	|	只读的，表示当前编译环境的ABI|
|TARGET_BOARD	|	只读的，表示当前编译环境的目标板|
|TARGET_VENDOR	|	只读的，表示当前编译环境的目标板厂商|
|==TARGET_WORKSPACE==	| 只读的，其值总是等于执行sys-build找到的第一个make.mk所在路径，如果在工程的顶层执行sys-build，则这个值等于工程路径，如果在子make中执行sys-build，其取值等于子make.mk所在路径。默认不需要定义它，如果需要强制指定路径，则可以在平台或板级配置文件中定义APP_WORKSPACE来修改。 |
|==TARGET_SDK_DIR==	|	只读的，表示工程总体编译时指向的工程的sdk包路径，它通常依赖于TARGET_WORKSPACE，注意，如果用户试图编译子make.mk文件，sys-build将会自动检查指向目录的有效性，如果未定义，将会给出一个警告。取值可以在平台或板级配置文件中定义APP_SDK_DIR来修改它，或者直接通过命令行传入。|
|==TARGET_LSP_DIR==	|	只读的，表示工程总体编译时指向的工程的lsp路径，它通常依赖于TARGET_WORKSPACE，注意，如果用户试图编译子make.mk文件，sys-build将会自动检查指向目录的有效性，如果未定义，将会给出一个警告。取值可以在平台或板级配置文件中定义APP_LSP_DIR来修改它，或者直接通过命令行传入。|
|==TARGET_RELEASE_DIR==	|	只读的，定义了对应平台的发布路径前缀，可以在平台或板级配置文件中定义APP_RELEASE_DIR来修改它，或者直接通过命令行传入。|
|APP_PROJECT_PATH	|	只读的，表示执行sys-build时找到的一个make.mk所在路径|
|TOOLCHAIN_NAME	|	只读的，当前编译器名字，如果模块定义了LOCAL_TARGET_TOOLCHAIN变量则无效|
|TOOLCHAIN_ROOT	|	只读的，当前编译器所在目录，如果模块定义了LOCAL_TARGET_TOOLCHAIN变量则无效|
|TARGET_OUT	|	只读的，表示当前编译系统生成所有未strip过的库和可执行程序的文件夹|
|NDK_APP_DST_DIR	|	只读的，表示当前编译系统生成的已经strip过的所有动态库和可执行文件目录|

上面表格中，加粗的变量仅用于编译第三方sdk或者内核、bootloader情况下，其他情况定义无效，高亮的变量是比较重要的变量。

### make.mk中能引用的函数

| 函数名                    | 说明                                                         | 用法                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| my-dir                    | 获取makefile最后一次包含文件的路径                           | v := $(call my-dir)                                          |
| this-makefile             | 返回当前makefile全路径                                       | curr_make := $(call this-makefile)                           |
| all-makefiles-under       | 返回指定目录下所有子目录的make.mk文件                        | subs_makes := $(call all-makefiles-under,../subs)            |
| all-subdir-makefiles      | 返回当前目录下所有子目录的make.mk文件                        | subs_makes := $(call all-subdir-makefiles)                   |
| include-makefile          | 包含指定路径下的make.mk文件                                  | $(call include-makefile,../subs/make.mk)                     |
| include-makefiles         | 包含多个make.mk列表                                          | $(call include-makefiles, make.mk ../make.mk)                |
| include-all-subs-makefile | 默认递归查找并包含指定目录下的所有make.mk文件                | $(call include-makefile,../subs1 ../subs2)或者$(call include-makefile,../subs1 ../subs2,make.mk) |
| get-all-wildcard-files    | 返回指定路径下所有匹配后缀的所有文件                         | $(call get-all-wildcard-files,subs src ,.c .cpp .cxx)        |
| get-all-archs             | 返回所有支持的架构                                           | $(call get-all-archs)                                        |
| get-all-platforms         | 返回指定架构下所有支持的平台                                 | $(call get-all-platforms,arm)                                |
| get-all-boards            | 返回指定架构和平台下所有支持的板子                           | $(call get-all-boards,arm,a5s)                               |
| get-all-abis              | 返回指定架构下所有支持的ABI                                  | $(call get-all-abis,arm)                                     |
| add-module-clean          | 添加一个自定义的清除目标支持，这个目标将被依赖到由第一个参数指定的clean目标上，如果第一个参数为空则默认被加入全局的clean依赖列表。 | $(call add-module-clean,,my_clean)或者$(call add-module-clean,module_name,my_clean) |



*******

# 使用sys-build编译工程

## 使用单个make.mk编译

一般我们对项目内某一模块的编译都是使用makefile进行编译的，不太会使用sys-build进行编译

### 编译普通可执行程序的make.mk

1、新建测试文件夹创建一个.c文件，命名为hello.c，编写一个hello world

```c
$ mkdir text
$ vi hello.c
    
#include <stdio.h>

int main(void)
{
    printf("hello world\n");
    return 0;
}
```

2、创建make.mk，编写make.mk文本

```makefile
$ vi make.mk
LOCAL_PATH := $(call my-dir)		# 定义当前LOCAL_PATH的值为makefile最后一次包含文件的路径

include $(CLEAR_VARS)				# 清除上次编译留下的残余变量值

LOCAL_MODULE := hello				# 定义模块名字，最终生成的名字就叫hello
LOCAL_SRC_FILES := hello.c			# 指定依赖的源文件
LOCAL_C_INCLUDES := \
        $(LOCAL_PATH)/../ \
        $(LOCAL_PATH)/../..			# 指定依赖的头文件路径（当不在同一个文件夹下）

include $(BUILD_EXECUTABLE)			# 将这个模块编译成可执行文件

```

3、sys-build编译，默认使用的是x86架构的release版本，你也可以根据你需求通过config跟换架构

```c
/home/jiwansu/tools/makerulers/build/core/setup-toolchain.mk:42: SYS-BUILD: "Invalid APP_X86_TOOLCHAIN defined in the `/home/jiwansu/tools/makerulers/build/core/default-application.mk`. using `gcc/g++(x86)` as default."    
Compile   : hello <= hello.c
Compile   : hello <= add.c
Executable     : hello
Install        : hello => out/libs/x86/x86/x86/default_board/hello
```

显示编译成功，生成的文件在out/libs/x86/x86/x86/default_board/目录下

### 编译bootloader模块

1、进入boot loader目录，先进行sys-build config配置相关目标板

```makefile
$ mkdir  -p ~/project/bootloader
$ cd ~/project/bootloader
$ vi make.mk
$ sys-build config hdu6_rk3568_rc_config
```

2、创建make.mk，编写make.mk，

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := boot
LOCAL_TARGET_TOP := ~/source/bootloader
#我们为内核单独指定编译器
LOCAL_TARGET_TOOLCHAIN := arm-linux-

#定义需要执行的目标，我们不需要make目标，直接make即可
#LOCAL_TARGET_RULER := 
#根据当前环境，确定boot的配置
ifeq ($(TARGET_BOARD),ipc2230)
LOCAL_TARGET_CONFIG := hdu6_rk3568_rc
else
LOCAL_TARGET_CONFIG := default
endif
#定义编译完成后需要复制的文件，该文件位于源码树的顶层目录
LOCAL_TARGET_COPY_FILES := u-boot.bin
#可以定义指定最终的release路径
#LOCAL_RELEASE_PATH ：= $(APP_PROJECT_PATH)/release
include $(BUILD_BOOTLOADER)
```

==注意==：这里我们发现在编写的时候发现需要指定编译工具链，深层次的原因不清楚，但是



## 集体编译整个工程

项目编译中，每一个工程的顶层都会有那么一个make.mk文件作为sys-build执行的入口。对这个make.mk文件的编写影响着整个工程的编译和模块的插入。

### 1、进入工程顶层目录

进行sys-build配置

```makefile
$ cd ~/project
$ sys-build config hdu6_rk3568_rc_config    
```

### 2、执行sys-build

在编译过程中，sys-build会根据不同目录下的make.mk完成相应的功能，将编译生成的文件放入指定文件夹中，如需查看位置可以去相应的目录下面的make.mk中进行查看，结束后会在顶层目录生成几个文件夹

```c
out/	release/	rootfs/
```

#### （1）out目录

```c
out
|--obj		//strip过的动态库和可执行程序的目录
|--lib    	//未strip过的动态库和可执行程序以及copy的文件和预编译的文件的目录
```

一般我们编译的镜像都不放在这个文件夹中了，现在都是放到release目录中去，这个文件夹我们不需要怎么关注

#### （2）release目录

这个目录是变量LOCAL_RELEASE_PATH指定的目录，如果是内核或boot或者第三方二进制模块，由变量LOCAL_TARGET_COPY_FILES指定的所有文件将被发布到这个目录中。

#### （3）rootfs目录

可以看作是一个中间文件，当在顶层目录执行sys-build makeos后，将会将这个目录打包压缩到release目录中的rootfs.tar.bz2中去，并且将该目录销毁

### 3、执行sys-build makeos

```c
$ sys-build makeos
```

执行这个命令，运行的是顶层目录中make.mk所引导到的main.mk中的makeos

```makefile
makeos:
ifeq ($(TARGET_BOARD), its200_rk3568)
	$(Q) cd $(TARGET_WORKSPACE)/rootfs && find . | cpio -H newc -o > $(TARGET_RELEASE_DIR)/boards/$(TARGET_PLATFORM)/$(TARGET_BOARD)/rootfs.cpio
	$(Q) cp $(TARGET_RELEASE_DIR)/boards/$(TARGET_PLATFORM)/$(TARGET_BOARD)/rootfs.cpio $(TARGET_WORKSPACE)/packages/linux_lsp/kernel/linux-4.19.193/
	$(Q) cd $(TARGET_WORKSPACE)/packages/linux_lsp/kernel/linux-4.19.193 && ./make_fit.sh $(TARGET_BOARD) $(APP_ARM64_TOOLCHAIN) $(TARGET_RELEASE_DIR)/boards/$(TARGET_PLATFORM)/$(TARGET_BOARD)
	$(Q) echo "Compile $(TARGET_PLATFORM)-$(TARGET_BOARD) finished"
else
	echo $(TMP_RELEASE_DIR)
	$(Q) cd $(TMP_RELEASE_DIR)&& ls -l && ./mksquashfs ${TARGET_WORKSPACE}/rootfs Image/rootfs.img
	$(Q) cd $(TARGET_WORKSPACE) && tar cvjf $(TMP_RELEASE_DIR)/rootfs.tar.bz2 rootfs
	$(Q) rm -rf $(ROOTFS_OUT)
	$(Q) cd $(TMP_RELEASE_DIR) && ./mkupdate.sh
	$(Q) cd $(TMP_RELEASE_DIR) && rm -rf mkupdate.sh
	$(Q) cd $(TMP_RELEASE_DIR) && rm -rf mksquashfs
	$(Q) cd $(TARGET_WORKSPACE)
	$(Q) echo "Compile $(TARGET_PLATFORM)-$(TARGET_BOARD) finished"
endif
```

由程序分析可知，这个命令将rootfs目录打包压缩到release目录中的rootfs.tar.bz2中去，并且将该目录销毁，之后由运行了几个脚本文件并删除，这样整个工程就编译完成了。

