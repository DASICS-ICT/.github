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

#### 2.1.2 编译bbl.bin

在riscv-pk仓库中：
```bash
make BOARD=xxx
# 仿真时BOARD=sim，上板时BOARD=pynq， 默认BOARD为sim
```
该仓库将自动完成rootfs和linux和bbl的编译步骤，在build文件夹内得到bbl.bin文件。

#### 2.1.3 进行纯仿真或difftest仿真

如果进行单纯仿真，则此时使用：

```bash
NEMU/build/riscv64-nemu-interpreter -b riscv-pk/build/bbl.bin
```

如果进行difftest测试，则此时在NutShell-DASICS仓库内进行`make emu`，并使用：

```bash
NutShell-DASICS/build/emu -i riscv-pk/build/bbl.bin
```

可以参照verilator的使用方法，使用`--enable-fork`功能dump波形，以便仿真调试硬件的bug。

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

在进行仿真/上板之间的切换时，如果出现问题，可以尝试`make clean`之后再次`make`。

（以上内容更新于dasics-nutshell-v2.2.2）
