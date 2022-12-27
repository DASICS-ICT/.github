# DASICS介绍

* DASICS(Dynamic in-Address-Space Isolation by Code Segments)是一种安全处理器设计方案，通过对不同代码片段访问的内存地址空间进行隔离并设置各自的访存权限，从而实现对非预期的越界访存和跳转的防护。这类越界访存可能来自于包括第三方恶意代码, 软件bug, 利用猜测执行的（如Spectre）漏洞在内的各种情况。

* 详情请查看我们提供的[DASICS白皮书]()

* 使用方式可以访问下载pdf版本的[用户手册](https://github.com/DASICS-ICT/DASICS-DOC)

# 现有仓库介绍
* 我们实现了对RISC-V架构Linux内核的修改，以支持DASICS相关安全处理机制，主要修改在如下几个仓库：
  * [riscv-linux](https://github.com/DASICS-ICT/riscv-linux)
  * [riscv-pk](https://github.com/DASICS-ICT/riscv-pk)
  * [riscv-rootfs](https://github.com/DASICS-ICT/riscv-rootfs)

* 我们实现了[QEMU的DASICS版本](https://github.com/DASICS-ICT/QEMU-DASICS)

* 我们在开源的RISC-V处理器[NutShell](http://https://github.com/OSCPU/NutShell)上实现了DASICS的硬件原型,并成功在FPGA上启动Linux并进行简单的安全测试
  * 支持DASICS功能的NutShell在[这里](https://github.com/DASICS-ICT/NutShell-DASICS)

#使用教程

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

* riscv-rootfs是制作内存文件系统使用的仓库，这个目录下面

## NutShell-DASICS PYNQ-Z2启动教程
