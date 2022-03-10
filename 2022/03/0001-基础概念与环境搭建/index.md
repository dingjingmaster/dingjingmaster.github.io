# imx6ull pro 基础概念与环境搭建


## 环境搭建准备

#### pc 和 配置目标
  - 通过`串口`可以用 PC 连接到 开发板，查看输出日志
  - 通过`ssh`可以让 PC 连接到 开发板，并对开发板进行控制和数据交换
    > 开发板默认登录用户名: `root`，无密码

#### pc 开发环境配置与准备
  - arm 交叉编译工具链环境 <重要>
  - 开发板引导程序源码  <后续可以准备>
  - 开发板linux内核源码 <后续可以准备>
  - imx6ull 要使用的文件系统，比如：busybox、buildroot、yocto  <后续可以准备>

> 我使用 `manjaro` 作为开发环境；使用 `imx6ull pro` 作为运行/学习环境，开发板下载的工具里没有linux平台的，因此需要从下载资料的 ubuntu 虚拟机里提取。

### 1. 先展示开发板
![开发板](/pic/imx6ull/000-banzi.jpg)
各标号硬件含义
![开发板](/pic/imx6ull/001-banzi.jpg)

### 2. 开发板启动方式（上图表号16对应状态）
|boot|`sw1(lcd_data5)`|`sw2(lcd_data11)`|`sw3(boot_mode0)`|`sw4(boot_mode1)`|
|---|---|---|---|---|
|emmc|OFF|OFF|ON|OFF|
|sd|ON|ON|ON|OFF|
|usb|X|X|OFF|ON|

> 注意：当设为 USB 启动时候，不能插上SD卡、TF卡；上电之后才可以插卡。刚出厂的板子在 emmc 上烧写了系统，开发板启动方式需要设置为 emmc 启动。

### 3. 第一次启动开发板
  1. 设置开发板的打开方式为 emmc
  2. 下载 linux 串口驱动程序 [`https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers`](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) 并编译之后在源码目录执行`insmod ./xxx.ko`，插入编译生成的内核模块(xxx.ko文件)
  3. 连接开发板电源线并打开开关，插拔 usb 线观察 `/dev/` 下设备变化，发现插入 usb 后会多出 `ttyUSB0` 这一设备
  4. 执行 `pacman -S minicom` 下载 minicom 
  5. 打开串口 `minicom -D /dev/ttyUSB0` 然后重启开发板(直接断点和上电)，后续就可以通过 minicom 看到串口日志了（需要开发板默认打开串口输出）

### 4. 开发环境搭建
  1. 给开发板联网并重启，在`系统`选项里设置网络(ip、子网掩码、网关)并用 PC ssh 连接上去
  2. 接上串口，用 `minicom` 来观察日志输出

## 环境配置与工具下载

> 开发环境是在pc上配置，在pc上使用交叉编译将代码编译成可执行文件，然后使用 ssh 发送到开发板，再在开发板上运行

### 1. 配置 make 环境
使用包管理器安装 `make` 包: `pacman -S make`

### 2. 配置 arm gcc 交叉编译环境(使用imx6ull资料配置)
> 补充：上述配置的 交叉编译环境链可能并不能用，需要使用imx6ull资料里提供的 交叉编译链，具体操作如下
1. 解压缩 ubuntu 的虚拟机文件（如果压缩文件是多个，需要都选中然后解压）
2. 查看解压后的文件有几个 `*.vmdk` 文件，如果是多个则需要使用 `vdiskmanager` 合并为一个
3. 使用 `qemu-img` 把 `xxx.vmdk` 文件转为 `qemu` 支持的 `qcow2` 文件
    > 我的 `linux` 工作机用 `qemu` 而不是 `vmware`
4. 挂载转换后的文件系统
5. 使用`mount`挂载虚拟硬盘的家目录，大概是 `/dev/nb0p4` 挂载后可以通过 `ls` 命令看到`book`文件夹，这就是家目录，然后把文件 `100ask_imx6ull-sdk` 复制出来，在这个文件夹的`ToolChain`目录下有需要的arm编译链
> 上述2、3、4步相关教程可以查看[qcow2虚拟分区挂载](/2022/03/0001-宿主机挂载qemu虚拟盘qcow2/)

### 2. 配置arm gcc交叉编译环境(这个留备后续使用，不需要配置)
1. arm 交叉编译工具下载地址: [arm 交叉编译工具下载地址](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/downloads)，或者复制到浏览器下载(这个包宿主机是x86，目标代码编译成arm的): `https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu/11.2-2022.02/binrel/gcc-arm-11.2-2022.02-x86_64-arm-none-linux-gnueabihf.tar.xz`
2. 下载并解压之后放置指定目录并改名为 `gcc-arm`，比如: `/data/envrionment/gcc-arm/` 
3. 配置环境变量，编辑 `~/.bashrc`，在末尾加入如下代码:
```shell
my_arm_home='/data/environment/gcc-arm'
my_path=$PATH
my_path=${my_arm_home}/bin:${my_path}
export PATH=$(echo ${my_path} | sed 's/:/\n/g' | sort | uniq | tr -s '\n' ':' | sed 's/:$//g')
```
4. 配置好后重新打开终端或执行`source ~/.bashrc` 之后打开终端，运行 `arm-none-linux-gnueabihf-gcc -v`，查看是否有如下信息输出(如果没有，重新执行这一步):
```shell
Using built-in specs.
COLLECT_GCC=arm-none-linux-gnueabihf-gcc
COLLECT_LTO_WRAPPER=/data/environment/gcc-arm/bin/../libexec/gcc/arm-none-linux-gnueabihf/11.2.1/lto-wrapper
Target: arm-none-linux-gnueabihf
Configured with: /data/jenkins/workspace/GNU-toolchain/arm-11/src/gcc/configure --target=arm-none-linux-gnueabihf --prefix= --with-sysroot=/arm-none-linux-gnueabihf/libc --with-build-sysroot=/data/jenkins/workspace/GNU-toolchain/arm-11/build-arm-none-linux-gnueabihf/install//arm-none-linux-gnueabihf/libc --with-bugurl=https://bugs.linaro.org/ --enable-gnu-indirect-function --enable-shared --disable-libssp --disable-libmudflap --enable-checking=release --enable-languages=c,c++,fortran --with-gmp=/data/jenkins/workspace/GNU-toolchain/arm-11/build-arm-none-linux-gnueabihf/host-tools --with-mpfr=/data/jenkins/workspace/GNU-toolchain/arm-11/build-arm-none-linux-gnueabihf/host-tools --with-mpc=/data/jenkins/workspace/GNU-toolchain/arm-11/build-arm-none-linux-gnueabihf/host-tools --with-isl=/data/jenkins/workspace/GNU-toolchain/arm-11/build-arm-none-linux-gnueabihf/host-tools --with-arch=armv7-a --with-fpu=neon --with-float=hard --with-mode=thumb --with-arch=armv7-a --with-pkgversion='GNU Toolchain for the Arm Architecture 11.2-2022.02 (arm-11.14)'
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 11.2.1 20220111 (GNU Toolchain for the Arm Architecture 11.2-2022.02 (arm-11.14))
```
> 这里需要注意的是，如果下载的 arm 编译工具链与开发板文件系统的编译工具链 gcc 不一致，则会导致在 pc 上用跨平台编译工具链编译出来的程序无法在 arm 开发板上运行。解决办法：1. 重新编译开发板根文件系统、内核并烧写；2. 使用官方提供的交叉编译工具链

### 3. cortexA7 烧写工具
> imx6ull 目前在 windows 上有带界面的烧写工具。linux 下暂时没找到相关工具

### 4. mkimage 工具
> 这一工具来源于 u-boot，用来给一个 bin 文件添加头部信息，芯片固件需要根据头部信息把 bin 文件放到内存中去执行
执行 `pacman -S u-boot` 命令后，再次执行 `mkimage -h` 查看是否正确安装:
```shell
mkimage: invalid option -- 'h'
Error: Invalid option
Usage: mkimage -l image
          -l ==> list image header information
       mkimage [-x] -A arch -O os -T type -C comp -a addr -e ep -n name -d data_file[:data_file...] image
          -A ==> set architecture to 'arch'
          -O ==> set operating system to 'os'
          -T ==> set image type to 'type'
          -C ==> set compression type 'comp'
          -a ==> set load address to 'addr' (hex)
          -e ==> set entry point to 'ep' (hex)
          -n ==> set image name to 'name'
          -d ==> use image data from 'datafile'
          -x ==> set XIP (execute in place)
       mkimage [-D dtc_options] [-f fit-image.its|-f auto|-F] [-b <dtb> [-b <dtb>]] [-E] [-B size] [-i <ramdisk.cpio.gz>] fit-image
           <dtb> file is used with -f auto, it may occur multiple times.
          -D => set all options for device tree compiler
          -f => input filename for FIT source
          -i => input filename for ramdisk file
          -E => place data outside of the FIT structure
          -B => align size in hex for FIT structure and header
Signing / verified boot options: [-k keydir] [-K dtb] [ -c <comment>] [-p addr] [-r] [-N engine]
          -k => set directory containing private keys
          -K => write public keys to this .dtb file
          -G => use this signing key (in lieu of -k)
          -c => add comment in signature node
          -F => re-sign existing FIT image
          -p => place external data at a static position
          -r => mark keys used as 'required' in dtb
          -N => openssl engine to use for signing
       mkimage -V ==> print version information and exit
Use '-T list' to see a list of available image types
```

## 最后来一个例子
### 在开发机上编写一个例子

```shell
#include <stdio.h>

int main (int argc, char* argv[])
{
    printf ("Hello World!\n");

    return 0;
}
```

### 开发机上的Makefile
```shell
CC = arm-linux-gnueabihf-gcc
CUR_DIR = $(shell pwd)
SRC = $(strip $(subst $(CUR_DIR), ., $(wildcard $(CUR_DIR)/*.c)))
TARGET = $(strip $(patsubst %.c, %.run, $(SRC)))

all:${TARGET}

%.run:%.c
        ${CC} -o $@ $<

clean:
        rm -f *.run
        rm -f *.o
```

### 编译
分别保存上述代码为 hello-world.c、makefile脚本为 Makefile，两个文件放在同级目录，执行 `make` 会在当前目录下生成 `hello-world.run`，使用 scp ./hello-world.run root@<开发机ip>:~ 把编译好的二进制文件传输到开发机，最后使用 ssh 登录到开发机上家目录执行 `./hello-world.run` 即可看到输出结果。

至此，开发环境和开发板打通。

