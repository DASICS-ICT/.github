# dasics-nutshell-v2 nemu仿真/pynq上板教程

本教程进行dasics-nutshell-v2的bbl + linux 4.18 nemu仿真/pynq上板。

## 1. 准备香山工具链
```bash
git clone git@github.com:OpenXiangShan/riscv-gnu-toolchain.git
```
编译riscv64-unknown-linux-gnu与riscv64-unknown-elf工具链作为dasics编译使用的工具链。

## 2. 准备DASICS-ICT组织的仓库

使用[dasics-nutshell-env](https://github.com/DASICS-ICT/dasics-nutshell-env)集成环境仓库进行简易部署。

## 3. 进行仿真/上板

### 3.1 仿真

由于dasics-nutshell-v2的仿真使用NEMU，故与DASICS-ICT组织主页的教程有一定差异；由于测试文件放在initramfs中，所以不需要准备额外的img作为文件系统。

#### 3.1.0 参照DASICS-ICT组织主页的教程设置$RISCV与$PATH等环境变量。
```bash
#假设你的dasics-riscv-toolchain工具链路径为$(DASICS_TOOL_CHAIN)
export RISCV=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-elf/bin:$PATH
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu/bin:$PATH
export RISCV_ROOTFS_HOME=$(PATH_TO_YOUR_RISCV_ROOTFS)
```

#### 3.1.1 编译NEMU

在NEMU仓库中：

```bash
make xxx_defconfig
make
# 使用riscv64-nutshell-dasics_defconfig（纯跑nemu）
# 或使用riscv64-nutshell-dasics-ref_defconfig （difftest）
```

#### 3.1.2 编译riscv-rootfs

在riscv-rootfs仓库的makefile内调整希望编译的内容，并使用make all编译。修改rootfsimg/initramfs-xxx.txt的内容可以根据自己的意愿调整根目录文件结构。

#### 3.1.3 修改riscv-linux配置

在riscv-linux仓库中： 

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- xxx_defconfig
# 仿真使用emu_defconfig，上板使用zynq_dasics_defconfig
```

可以修改对应defconfig文件的CONFIG_INITRAMFS相关内容指向riscv-rootfs仓库内不同的initramfs-xxx.txt，以使用自定义的根目录文件结构。

#### 3.1.4 编译riscv-linux与riscv-pk

在riscv-pk仓库中make，在build文件夹内得到bbl.bin文件。

每次编译riscv-linux时会出现 multiple definition of 'yylloc' 的错误，需要修改riscv-linux/scripts/dtc/dtc-lexer.lex.c，在YYLTYPE yylloc前加上extern。

注意！仓库内默认代码为pynq上板时使用，如果仿真使用需要先参照备注进行部分代码修改！

#### 3.1.5 进行纯仿真或difftest仿真

如果进行单纯仿真，则此时使用：

```bash
NEMU/build/riscv64-nemu-interpreter -b riscv-pk/build/bbl.bin
```

如果进行difftest测试，则此时在NutShell-DASICS仓库内进行make emu，并使用：

```bash
NutShell-DASICS/build/emu -i riscv-pk/build/bbl.bin
```

### 3.2 上板

使用vivado 2019.2，按照DASICS-ICT组织主页的教程上板；唯一的不同是，不需要再单独修改Kconfig的ZYNQ_ONBOARD，在make linux kernel时使用对应的defconfig即可。

### 备注

在nemu仿真与zynq上板之间切换时需要进行一些代码修改，如下：

在riscv-pk仓库内修改:

dts/system.dts的include：上板为zynq-standalone.dtsi；仿真为noop.dtsi(platform.dtsi)

Makefile的BBL_CONFIG --with-mem-start选项和bbl/bbl.mk.in的bbl.bin --change-addresses选项；上板为0x50000000；仿真为0x80000000
        
在进行仿真/上板之间的切换时记得先clean再make；每次clean后编译Linux都需要进行extern yylloc的修改。
