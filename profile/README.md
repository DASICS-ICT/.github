# 一、DASICS介绍

* [README in English](https://github.com/DASICS-ICT/.github/blob/main/profile/README-en.md)

* DASICS(Dynamic in-Address-Space Isolation by Code Segments)是一种安全处理器设计方案，通过对不同代码片段访问的内存地址空间进行隔离并设置各自的访存权限，从而实现对非预期的越界访存和跳转的防护。这类越界访存可能来自于包括第三方恶意代码, 软件bug, 利用猜测执行的（如Spectre）漏洞在内的各种情况。

* DASICS设计思想和具体实现请查看我们提供的[DASICS白皮书](https://github.com/DASICS-ICT/DASICS-DOC/blob/main/doc/DASICS_white_paper.pdf)

* 使用方式可以访问下载pdf版本的[用户手册](https://github.com/DASICS-ICT/DASICS-DOC/blob/main/doc/DASICS_user_manual.pdf)

* 相关学术论文和资料：
  * Zhao Y Y, Chen M Y, Liu Y H, et al. IMPULP: A Hardware Approach for In-Process Memory Protection via User-Level Partitioning[J]. Journal of Computer Science and Technology, 2020, 35(2): 418-432.
  * [第二届中国峰会上DASICS的报告](https://www.bilibili.com/video/BV1CG41157qu/?spm_id_from=333.337.search-card.all.click)

# 二、现有仓库介绍
* 我们实现了对RISC-V架构Linux内核的修改，以支持DASICS相关安全处理机制，主要修改在如下几个仓库：
  * [riscv-linux](https://github.com/DASICS-ICT/riscv-linux)
  * [riscv-pk](https://github.com/DASICS-ICT/riscv-pk)
  * [riscv-rootfs](https://github.com/DASICS-ICT/riscv-rootfs)

* 我们实现了[QEMU的DASICS版本](https://github.com/DASICS-ICT/QEMU-DASICS)

* 我们在开源的RISC-V处理器[NutShell](http://https://github.com/OSCPU/NutShell)上实现了DASICS的硬件原型,并成功在FPGA上启动Linux并进行简单的安全测试
  * 支持DASICS功能的NutShell在[这里](https://github.com/DASICS-ICT/NutShell-DASICS)

* 我们实现了基础的DASICS测试，测试代码build目录在`riscv-rootfs/apps/dasics-test`，目前有四个基本的测试：
  * dasics-test-ofb：测试非可信区函数对越界地址进行读/写/跳转
  * dasics-test-jump：测试可信区、非可信区以及自由跳转区之间的跳转功能
  * dasics-test-rwx：测试非可信区边界寄存器的分配和设置，以及读/写/执行权限是否能正常限制
  * dasics-test-free：测试非可信区边界控制寄存器是否能正常释放
  * dasics-test-syscall：测试非可信区write syscall是否能被可信区拦截以及正常代理
```
# riscv-rootfs/apps/dasics-test
test
├── dasics-test-free.c
├── dasics-test-jump.c
├── dasics-test-ofb.c
├── dasics-test-rwx.c
└── dasics-test-syscall.c
```


* 下面详细说明了[如何在qemu模拟器上](#qemu) 跑支持DASICS功能的linux以及上述测试，以及[如何在FPGA上](#pynq)使用支持DASICS功能的NutShell处理器核运行linux以及测试

# 三、使用教程

<h3 id=qemu ></h3>

参照xs-dasics-v1.0用户手册更新后的dasics-nutshell-v2已经发布。

[dasics-nutshell-v2使用教程](https://github.com/DASICS-ICT/.github/blob/main/profile/dasics-nutshell-v2.md)

本文以下为dasics-nutshell-v1的使用教程。

## 3.1 QEMU使用教程

这个教程主要指导如何在QEMU模拟器上把我们提供的支持DASICS功能的Linux跑起来，并进行一些基础的测试。

### 3.1.1 直接使用Release环境包

* 如果不想进行下面的操作，可以直接使用我们提供的开箱即用的[dasics-qemu 环境包](https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0.0/dasics-qemu.tar.gz)，这里面包含了按照下面的骤准备好的各个文件,但由于可能缺少库文件会导致出错，因此我们还是建议按照下面的步骤准备相关文件.

* 解压后，进入`dasics-qemu目录`，运行如下命令：
```
./qemu-system-riscv64 -machine virt -bios none -kernel ~/qemu-test/dasics/bbl -m 1G -nographic -append "console=ttyS0 rw root=/dev/vda" -drive file=../img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0
```

* 成功运行将会打印出linux启动信息

### 3.1.2 准备QEMU

* 首先把我们提供的QEMU-DASICS clone到本地
``` 
git clone https://github.com/DASICS-ICT/QEMU-DASICS.git
```

* 之后对对QEMU进行编译的到`qemu-system-riscv64`可执行文件
``` 
cd QEMU-DASICS
./configure --target-list=riscv64-softmmu
make clean && make
```

* 生成`qemu-system-riscv64`在`riscv64-softmmu`目录下,接下来我们需要制作一个virtio盘,在`QEMU-DASIC`目录下执行下列命令：
```
./qemu-img create -f raw img 1G
mkfs.ext4 img
sudo mount img tmp_mount/
```

* 上面最后一步是为了把 `img` mount到一个文件夹`tmp_mnt`上，然后进`tmp_mnt`修改就可以修改`img`里的内容。

* 到此QEMU的准备完成

### 3.1.3 准备linux bbl

* 首先要确认有riscv编译工具链，如果没有可以从[源码制作](https://github.com/riscv-collab/riscv-gnu-toolchain),注意最好使用11.1.0版本的gcc,也可以使用我们在release中提供的[预编译包](https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0.0/dasics-riscv-toolchain.tar.gz)，下面我们按照预编译包的方式进行讲解：

* 解压`dasics-riscv-toolchain.tar.gz`，在`dasics-riscv-toolchain`目录下有`riscv64-unknown-elf`和`riscv64-unknown-linux-gnu`两个目录

* 设置好`RISCV`变量并将工具链添加到环境变量中，假设你的`dasics-riscv-toolchain`工具链路径为`$(DASICS_TOOL_CHAIN)`,运行如下命令(（建议添加到`~/.bashrc`或者`~/.zshrc`中)：
```
export RISCV=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-elf/bin:$PATH
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu/bin:$PATH
```

* 工具链测试，运行如下命令，如果没有错误提示正常显示gcc信息则说明工具链安装正确，我们需要两个工具链，包括`riscv64-unknown-elf-`和`riscv64-unknown-linux-gnu-`，下列命令都应该显示是11.1.0 版本的gcc
```
riscv64-unknown-linux-gnu-gcc -v
riscv64-unknown-elf-gcc -v
```

* 接下来将我们提供的`riscv-linux`、`riscv-pk`、`riscv-rootfs`仓库clone下来
```
git clone https://github.com/DASICS-ICT/riscv-linux.git
git clone https://github.com/DASICS-ICT/riscv-rootfs.git
git clone https://github.com/DASICS-ICT/riscv-pk.git
```

* 设置好`RISCV_ROOTFS_HOME`,为`riscv-rootfs`的路径（建议添加到`~/.bashrc`或者`~/.zshrc`中）,假设是`$(PATH_TO_YOUR_RISCV_ROOTFS)`
```
export RISCV_ROOTFS_HOME=$(PATH_TO_YOUR_RISCV_ROOTFS)
```

* `riscv-rootfs`是制作内存文件系统使用的仓库，这个目录下面`apps/busybox`是我们需要的一些busybox基本工具，`apps/dasics-test`是我们dasics测试的build目录。我们在`riscv-rootfs`目录下运行
```
make all
```

* 成功后，我们已经在`riscv-rootfs/rootfsimg`目录下创建好了内存文件系统，文件系统描述见`riscv-rootfs/rootfsimg/initramfs-dasics.txt`文件

* 接下来我们进入`riscv-linux`编译linux内核，需要注意的是，如果我们是在QEMU上跑linux，需要进入`riscv-linux/arch/riscv/Kconfig`, 在105-108行将`CONFIG_ZYNQ_ONBOARD`设置为`n`
```
//riscv-linux/arch/riscv/Kconfig
config ZYNQ_ONBOARD
#	   def_bool y	
     def_bool n
```

* 修改好后在`riscv-linux`目录下运行下述命令（`qemu_defconfig`文件在`riscv-linux/arch/riscv/configs`下）
```
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- qemu_defconfig
```

* 最后我们进入`riscv-pk`目录下运行下述命令，成功之后在`riscv-pk/build`目录下得到我们的`bbl`文件，这个文件包含了boot loader、linux kernel以及我们制作的内存文件系统
```
make qemu
```

### 3.1.4 在QEMU上运行linux bbl

* 回到`QEMU-DASICS`目录运行下述命令(假设我们上一步中得到的bbl路径为`$(PATH_TO_BBL)`：
```
cd riscv64-softmmu
./qemu-system-riscv64 -machine virt -bios none -kernel $(PATH_TO_BBL) -m 1G -nographic -append "console=ttyS0 rw root=/dev/vda" -drive file=../img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0
```

* 等待linux启动直到进入shell，`/root`目录下放置了4个编译好的dasics测试，`/root/scripts/run-dasics-test.sh`是跑所有测试的脚本，执行如下命令：
```
sh root/scripts/run-dasics-test.sh
```

* 可以看到输出的信息中包含了4个测试的打印输出，说明dasics功能正确跑通

![](https://github.com/DASICS-ICT/.github/blob/main/qemu-linux.gif)

<h3 id=pynq ></h3>

## 3.2 NutShell-DASICS PYNQ-Z2 FPGA启动教程

这个教程主要指导如何将我们修改的NutShell处理器核以及Linux镜像放到PYNQ-Z2 FPGA板上跑起来。

同样如果不想进行下述步骤，可以使用我们release里给出的[dasics-pynq包](https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0.0/dasics-pynq.tar.gz)，里面包含准备好的BOOT.BIN和RV_BOOT.bin，可以直接跳转到[上板步骤](#fpga-onboard)

### 3.2.1 准备工作

* 安装 [mill](https://com-lihaoyi.github.io/mill/mill/Intro_to_Mill.html)

* 确保python在$PATH中

* 安装 [Vivado 2019.2](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)

* 在 [PYNQ-Z2 页面](https://www.tulembedded.com/FPGA/ProductsPYNQ-Z2.html)上下载 PYNQ-Z2 Board File，并将其加入Vivado安装目录中的Vivado/2019.2/data/boards/board_files中

### 3.2.2 制作PYNQ板的内存镜像

这一步基本和qemu中类似，因此就简述一下差别

* 需要和qemu一样先`export RISCV`变量和`RISCV_ROOTFS_HOME`变量

* 在`riscv-rootfs`中运行`make all`

* 查看`riscv-linux/arch/riscv/Kconfig`, `CONFIG_ZYNQ_ONBOARD`需要设置为`y`

* 在riscv-linux中运行`make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- zynq_dasics_defconfig `

* 在riscv-pk中运行`make`，生成的bbl.bin 在riscv-pk/build文件夹下，将bbl.bin改名为RV_BOOT.bin

### 3.2.3 生成verilog

* 获取NutShell-DASICS仓库，进入dev-dasics-ucas-os分支
```
git clone https://github.com/DASICS-ICT/NutShell-DASICS.git
cd NutShell-DASICS
git checkout dev-dasics-ucas-os
```
* 运行以下命令：
```
make BOARD=pynq
```

### 3.2.4 用vivado工程生成启动镜像

* 确保vivado在$PATH中（可能需要source Vivado安装目录下的Vivado/2019.2/settings64.sh）
 
* 将fpga/board/run.tcl第2行和第6行的-jobs 20修改为合适的并发数
 
* 在NutShell-DASICS所在目录下获取 [Xilinx/device-tree-xlnx](https://github.com/Xilinx/device-tree-xlnx) 仓库，并切换到xilinx-v2019.2 tag
 
* 运行以下命令生成启动镜像：
```
cd fpga && make PRJ=prj BOARD=pynq STANDALONE=true bootgen
```
* 成功运行后，生成fpga/boot/build/prj-pynq/BOOT.BIN文件

<h3 id=fpga-onboard ></h3>

### 3.2.5 上板

* 准备SD卡，对SD卡进行分区，并对SD卡第一个分区进行格式化

* 将BOOT.BIN和RV_BOOT.bin拷贝到SD卡的第一个分区

* 板卡上电之后可以看到linux启动，其余步骤和qemu测试相同
