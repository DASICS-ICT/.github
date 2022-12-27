# DASICS介绍

* DASICS(Dynamic in-Address-Space Isolation by Code Segments)是一种安全处理器设计方案，通过对不同代码片段访问的内存地址空间进行隔离并设置各自的访存权限，从而实现对非预期的越界访存和跳转的防护。这类越界访存可能来自于包括第三方恶意代码, 软件bug, 利用猜测执行的（如Spectre）漏洞在内的各种情况。

* DASICS设计思想和具体实现请查看我们提供的[DASICS白皮书]()

* 使用方式可以访问下载pdf版本的[用户手册](https://github.com/DASICS-ICT/DASICS-DOC)

* 相关学术论文和资料：
  * Zhao Y Y, Chen M Y, Liu Y H, et al. IMPULP: A Hardware Approach for In-Process Memory Protection via User-Level Partitioning[J]. Journal of Computer Science and Technology, 2020, 35(2): 418-432.
  * [第二届中国峰会上DASICS的报告](https://www.bilibili.com/video/BV1CG41157qu/?spm_id_from=333.337.search-card.all.click)

# 现有仓库介绍
* 我们实现了对RISC-V架构Linux内核的修改，以支持DASICS相关安全处理机制，主要修改在如下几个仓库：
  * [riscv-linux](https://github.com/DASICS-ICT/riscv-linux)
  * [riscv-pk](https://github.com/DASICS-ICT/riscv-pk)
  * [riscv-rootfs](https://github.com/DASICS-ICT/riscv-rootfs)

* 我们实现了[QEMU的DASICS版本](https://github.com/DASICS-ICT/QEMU-DASICS)

* 我们在开源的RISC-V处理器[NutShell](http://https://github.com/OSCPU/NutShell)上实现了DASICS的硬件原型,并成功在FPGA上启动Linux并进行简单的安全测试
  * 支持DASICS功能的NutShell在[这里](https://github.com/DASICS-ICT/NutShell-DASICS)

* 我们实现了基础的DASICS测试，测试代码目录在

# 使用教程

## QEMU使用教程
这个教程主要指导如何在QEMU模拟器上把我们提供的支持DASICS功能的Linux跑起来，并进行一些基础的测试。如果不想进行下面的操作，可以直接使用我们提供的开箱即用的[环境包]，这里面包含了

### 准备QEMU

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

* 生成`qemu-system-riscv64`在`riscv64-softmmu`目录下,接下来我们需要制作一个virtio盘,在QEMU-DASIC目录下执行下列命令：
```
./qemu-img create -f raw img 1G
mkfs.ext4 img
sudo mount img tmp_mount/
```

* 上面最后一步是为了把 `img` mount到一个文件夹`tmp_mnt`上，然后进`tmp_mnt`修改就可以修改`img`里的内容。

* 到此QEMU的准备完成

### 准备linux bbl

* 首先要确认有riscv编译工具链，如果没有可以从[源码制作](https://github.com/riscv/riscv-tools),也可以使用NutShell原本提供的[预编译包](https://github.com/LvNA-system/labeled-RISC-V/releases/download/v0.1.0/riscv-toolchain-2018.05.24.tar.gz)

* 设置好RISCV变量并将工具链添加到环境变量中，假设你的工具链路径为$(TOOL_CHAIN),运行如下命令：
```
export RISCV=$(TOOL_CHAIN)
export PATH=$(TOOL_CHAIN)/bin:$PATH
```

* 工具链测试，运行如下命令，如果没有错误提示正常显示gcc信息则说明工具链安装正，我们需要两个工具链，包括`riscv64-unknown-elf-`和`riscv64-unknown-linux-gnu-`
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

* 设置好RISCV_ROOTFS_HOME,为riscv-rootfs的路径（建议添加到~/.bashrc或者~/.zshrc中）,假设是$(PATH_TO_YOUR_RISCV_ROOTFS)
```
export RISCV_ROOTFS_HOME=$(PATH_TO_YOUR_RISCV_ROOTFS)
```

* riscv-rootfs是制作内存文件系统使用的仓库，这个目录下面apps/busybox是我们需要的一些busybox基本工具，apps/dasics-test是我们dasics测试的build目录。我们在riscv-rootfs目录下运行
```
make all
```

* 成功后，我们已经在riscv-rootfs/rootfsimg目录下创建好了内存文件系统，文件系统描述见riscv-rootfs/rootfsimg/initramfs-dasics.txt文件

* 接下来我们进入riscv-linux编译linux内核，需要注意的是，如果我们是在QEMU上跑linux，需要进入riscv-linux/arch/riscv/Kconfig, CONFIG_ZYNQ_ONBOARD需要设置为n
```
//riscv-linux/arch/riscv/Kconfig 105-108
config ZYNQ_ONBOARD
#	   def_bool y	
     def_bool n
```

* 修改好后在riscv-linux目录下运行下述命令（qemu_defconfig文件在riscv-linux/arch/riscv/configs下）
```
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- qemu_defconfig
```

* 最后我们进入riscv-pk目录下运行下述命令，之后在riscv-pk/build目录下得到我们的bbl文件，这个文件包含了boot loader、linux kernel以及我们制作的内存文件系统

### 在QEMU上运行linux bbl

* 回到QEMU-DASICS目录运行下述命令(假设我们上一步中得到的bbl路径为$(PATH_TO_BBL))：
```
cd riscv64-softmmu
./qemu-system-riscv64 -machine virt -bios none -kernel $(PATH_TO_BBL) -m 1G -nographic -append "console=ttyS0 rw root=/dev/vda" -drive file=../img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0
```

* 等待linux启动直到进入shell，/root目录下放置了4个编译好的dasics测试，/root/scripts/run-dasics-test.sh是跑所有测试的脚本，执行如下命令：
```
sh root/scripts/run-dasics-test.sh
```

* 可以看到输出的信息中包含了4个测试的打印输出，说明dasics功能正确跑通

## NutShell-DASICS PYNQ-Z2启动教程

### 准备工作
* 安装 [mill](https://com-lihaoyi.github.io/mill/mill/Intro_to_Mill.html)
* 确保python在$PATH中
* 安装 [Vivado 2019.2](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)
* 在 [PYNQ-Z2 页面](https://www.tulembedded.com/FPGA/ProductsPYNQ-Z2.html)上下载 PYNQ-Z2 Board File，并将其加入Vivado安装目录中的Vivado/2019.2/data/boards/board_files中

### 制作ZYNQ板的内存镜像

这一步基本和qemu中类似，因此就简述一下差别

* 需要和qemu一样先`export RISCV`变量和`RISCV_ROOTFS_HOME`变量

* 在`riscv-rootfs`中运行`make all`

* 查看`riscv-linux/arch/riscv/Kconfig`, `CONFIG_ZYNQ_ONBOARD`需要设置为`y`

* 在riscv-linux中运行`make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- zynq_dasics_defconfig `

* 在riscv-pk中运行`make`，生成的bbl.bin 在riscv-pk/build文件夹下，将bbl.bin改名为RV_BOOT.bin

### 生成verilog

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

### 用vivado工程生成启动镜像

* 确保vivado在$PATH中（可能需要source Vivado安装目录下的Vivado/2019.2/settings64.sh）
 
* 将fpga/board/run.tcl第2行和第6行的-jobs 20修改为合适的并发数
 
* 在NutShell-DASICS所在目录下获取 [Xilinx/device-tree-xlnx](https://github.com/Xilinx/device-tree-xlnx) 仓库，并切换到xilinx-v2019.2 tag
 
* 运行以下命令生成启动镜像：
```
cd fpga && make PRJ=prj BOARD=pynq STANDALONE=true bootgen
```
* 成功运行后，生成fpga/boot/build/prj-pynq/BOOT.BIN文件

### 上板
* 准备SD卡，对SD卡进行分区，并对SD卡第一个分区进行格式化

* 将BOOT.BIN和RV_BOOT.bin拷贝到SD卡的第一个分区

* 办卡上电之后可以看到linux启动，其余步骤和qemu测试相同
