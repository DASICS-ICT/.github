# dasics-nutshell-v2 nemu仿真/pynq上板教程

本教程进行dasics-nutshell-v2的bbl + linux 4.18 nemu仿真/pynq上板。

## 1. 准备DASICS-ICT组织的仓库

使用[dasics-nutshell-env](https://github.com/DASICS-ICT/dasics-nutshell-env)集成环境仓库进行简易部署。

## 2. 进行仿真/上板

### 2.1 仿真

由于dasics-nutshell-v2的仿真使用NEMU，故与DASICS-ICT组织主页的教程有一定差异；由于测试文件放在initramfs中，所以不需要准备额外的img作为文件系统。

#### 2.1.1 编译NEMU

在NEMU仓库中：

```bash
make xxx_defconfig
make
# 使用riscv64-nutshell-dasics_defconfig（纯跑nemu）
# 或使用riscv64-nutshell-dasics-ref_defconfig （difftest）
```

#### 2.1.2 编译riscv-rootfs

在riscv-rootfs仓库的makefile内调整希望编译的内容，并使用make all编译。修改rootfsimg/initramfs-xxx.txt的内容可以根据自己的意愿调整根目录文件结构。

#### 2.1.3 修改riscv-linux配置

在riscv-linux仓库中： 

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- xxx_defconfig
# 仿真使用emu_defconfig，上板使用zynq_dasics_defconfig
```

可以修改对应defconfig文件的CONFIG_INITRAMFS相关内容指向riscv-rootfs仓库内不同的initramfs-xxx.txt，以使用自定义的根目录文件结构。

#### 2.1.4 编译riscv-linux与riscv-pk

在riscv-pk仓库中make，在build文件夹内得到bbl.bin文件。

每次编译riscv-linux时会出现 multiple definition of 'yylloc' 的错误，需要修改riscv-linux/scripts/dtc/dtc-lexer.lex.c，在YYLTYPE yylloc前加上extern。

注意！仓库内默认代码为pynq上板时使用，如果仿真使用需要先参照备注进行部分代码修改！

#### 2.1.5 进行纯仿真或difftest仿真

如果进行单纯仿真，则此时使用：

```bash
NEMU/build/riscv64-nemu-interpreter -b riscv-pk/build/bbl.bin
```

如果进行difftest测试，则此时在NutShell-DASICS仓库内进行make emu，并使用：

```bash
NutShell-DASICS/build/emu -i riscv-pk/build/bbl.bin
```

### 2.2 上板

使用vivado 2020.2，按照DASICS-ICT组织主页的教程3.2.3-3.2.5上板；唯一的不同是，不需要再单独修改Kconfig的ZYNQ_ONBOARD，在make linux kernel时使用对应的defconfig即可。

```bash
cd NutShell-DASICS
make BOARD=pynq
cd fpga && make PRJ=prj BOARD=pynq STANDALONE=true bootgen
# 成功运行后，生成fpga/boot/build/prj-pynq/BOOT.BIN文件
# 将上板用的bbl.bin更名为RV_BOOT.bin，和BOOT.BIN拷贝到SD卡的第一个分区
```

### 备注

在nemu仿真与zynq上板之间切换时需要进行一些代码修改，如下：

在riscv-pk仓库内修改:

dts/system.dts的include：上板为zynq-standalone.dtsi；仿真为noop.dtsi(platform.dtsi)

Makefile的BBL_CONFIG --with-mem-start选项和bbl/bbl.mk.in的bbl.bin --change-addresses选项；上板为0x50000000；仿真为0x80000000
        
在进行仿真/上板之间的切换时记得先clean再make；每次clean后编译Linux都需要进行extern yylloc的修改。
