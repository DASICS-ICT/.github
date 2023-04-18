# 1. DASICS Introduction

* * [README in Chinese](https://github.com/DASICS-ICT/.github/blob/main/profile/README.md)

* DASICS (Dynamic in-Address-Space Isolation by Code Segments) is a secure processor solution that protects against unintended out-of-bounds accesses and bounces by isolating the memory address spaces accessed by different code segments and setting their respective access rights. Such out-of-bounds accesses can come from a variety of scenarios including third-party malicious code, software bugs, and exploiting guessed execution (e.g., Spectre) vulnerabilities.

* [DASICS White Paper](https://github.com/DASICS-ICT/DASICS-DOC/blob/main/doc/DASICS_white_paper.pdf) is available.

* [DASICS User Manual](https://github.com/DASICS-ICT/DASICS-DOC) is available.

* Related Academic Papers and Materials：
  * Zhao Y Y, Chen M Y, Liu Y H, et al. IMPULP: A Hardware Approach for In-Process Memory Protection via User-Level Partitioning[J]. Journal of Computer Science and Technology, 2020, 35(2): 418-432.
  * [DASICS presentation at the 2nd RISC-V China Summit](https://www.bilibili.com/video/BV1CG41157qu/?spm_id_from=333.337.search-card.all.click)

# 2. Introduction of Existing Repos
* We implemented modifications to the RISC-V architecture Linux kernel to support DASICS-related security processing mechanisms, mainly in the following repositories:
  * [riscv-linux](https://github.com/DASICS-ICT/riscv-linux)
  * [riscv-pk](https://github.com/DASICS-ICT/riscv-pk)
  * [riscv-rootfs](https://github.com/DASICS-ICT/riscv-rootfs)

* We implemented [the DASICS version of QEMU](https://github.com/DASICS-ICT/QEMU-DASICS)

* We implemented a hardware prototype of DASICS on the open source RISC-V processor [NutShell](http://https://github.com/OSCPU/NutShell), and successfully booted Linux on the FPGA and performed simple security tests.
  * NutShell with DASICS support is available [here](https://github.com/DASICS-ICT/NutShell-DASICS)

* We implemented the basic DASICS test, the test code build directory is in `riscv-rootfs/apps/dasics-test` and there are currently four basic tests：
  * dasics-test-ofb：Test non-trusted zone functions for read/write/jump to out-of-bounds addresses;
  * dasics-test-jump：Test the jump function between trusted, non-trusted and free jumping zones;
  * dasics-test-rwx：Test the allocation and setting of non-trusted zone boundary registers and whether read/write/execute permissions can be restricted properly;
  * dasics-test-free：Test that the non-trusted zone boundary control register is released correctly.
  * * dasics-test-syscall：Test whether write syscall in untrusted zones can be intercepted by trusted zones and proxied correctly.
```
# riscv-rootfs/apps/dasics-test
test
├── dasics-test-free.c
├── dasics-test-jump.c
├── dasics-test-ofb.c
├── dasics-test-rwx.c
└── dasics-test-syscall.c
```


* The following details [how to run DASICS-enabled linux on a qemu emulator](#qemu) and the above tests, and [how to run linux on an FPGA](#pynq) with a DASICS-enabled NutShell processor core and the tests.

# 3. Tutorials

<h3 id=qemu ></h3>

## 3.1 QEMU Usage Tutorial

This tutorial is a guide on how to get our provided DASICS-enabled Linux up and running on the QEMU emulator and perform some basic tests.

### 3.1.1 Use Release Environment Package

* If you don't want to do the following steps, you can use our out-of-the-box [dasics-qemu environment package](https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0), which contains the files prepared as described below, but we recommend that you prepare the files as described below because of possible errors caused by missing library files.

* After unpacking, go to the `dasics-qemu directory` and run the following command：
```
./qemu-system-riscv64 -machine virt -bios none -kernel ~/qemu-test/dasics/bbl -m 1G -nographic -append "console=ttyS0 rw root=/dev/vda" -drive file=../img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0
```

* A successful run will print out the linux boot information.

### 3.1.2 Prepare QEMU

* First clone the QEMU-DASICS we provide
``` 
git clone https://github.com/DASICS-ICT/QEMU-DASICS.git
```

* Afterwards, the QEMU is compiled to the `qemu-system-riscv64` executable
``` 
cd QEMU-DASICS
./configure --target-list=riscv64-softmmu
make clean && make
```

* You will get `qemu-system-riscv64` in the `riscv64-softmmu` directory. Next we need to make a virtio disk, in the `QEMU-DASIC` directory execute the following command:
```
./qemu-img create -f raw img 1G
mkfs.ext4 img
sudo mount img tmp_mount/
```

* The last step is to mount `img` to a folder `tmp_mnt`, then go into `tmp_mnt` and modify the contents of `img`.

### 3.1.3 Prepare Linux BBL

* First make sure there is a riscv compilation toolchain, if not you can get it from [source code production](https://github.com/riscv-collab/riscv-gnu-toolchain). Note that it is best to use version 11.1.0 of gcc, or you can use the [pre-compilation package](https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0.0/dasics-riscv-toolchain.tar.gz) we provide in the release. We use the pre-compiled package for description:

* Extract `dasics-riscv-toolchain.tar.gz`, there are two directories `riscv64-unknown-elf` and `riscv64-unknown-linux-gnu` in the `dasics-riscv-toolchain` directory

* Set up the `RISCV` variable and add the toolchain to the environment variable, assuming your `dasics-riscv-toolchain` toolchain path is `$(DASICS_TOOL_CHAIN)`, run the following command (recommended to add to `~/.bashrc` or `~/.zshrc`).
```
export RISCV=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-elf/bin:$PATH
export PATH=$(DASICS_TOOL_CHAIN)/riscv64-unknown-linux-gnu/bin:$PATH
```

* To test the toolchain, run the following command, if there is no error message and the gcc information is displayed normally then the toolchain is installed correctly, we need two toolchains including `riscv64-unknown-elf-` and `riscv64-unknown-linux-gnu-`, the following commands should both display the 11.1.0 version of gcc:
```
riscv64-unknown-linux-gnu-gcc -v
riscv64-unknown-elf-gcc -v
```

* Next, clone down the `riscv-linux`, `riscv-pk`, and `riscv-rootfs` repositories we provided:
```
git clone https://github.com/DASICS-ICT/riscv-linux.git
git clone https://github.com/DASICS-ICT/riscv-rootfs.git
git clone https://github.com/DASICS-ICT/riscv-pk.git
```

* Set `RISCV_ROOTFS_HOME` to the path of `riscv-rootfs` (we recommend adding it to `~/.bashrc` or `~/.zshrc`), assuming it is `$(PATH_TO_YOUR_RISCV_ROOTFS)`
```
export RISCV_ROOTFS_HOME=$(PATH_TO_YOUR_RISCV_ROOTFS)
```

* `riscv-rootfs` is the repository used to make the memory file system, under this directory `apps/busybox` is some of the basic tools we need for the busybox, and `apps/dasics-test` is the build directory for our dasics tests. We run in the `riscv-rootfs` directory:
```
make all
```

* After success, we have created the memory file system in the `riscv-rootfs/rootfsimg` directory, the file system description is in the `riscv-rootfs/rootfsimg/initramfs-dasics.txt` file

* Next we go to `riscv-linux` to compile the linux kernel, it should be noted that if we are running linux on QEMU, we need to go to `riscv-linux/arch/riscv/Kconfig`, set `CONFIG_ZYNQ_ONBOARD` to `n` at lines 105-108:
```
//riscv-linux/arch/riscv/Kconfig
config ZYNQ_ONBOARD
#	   def_bool y	
     def_bool n
```

* Run the following command in the `riscv-linux` directory after modification (the `qemu_defconfig` file is under `riscv-linux/arch/riscv/configs` directory)
```
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- qemu_defconfig
```

* Finally we go to the `riscv-pk` directory and run the following command, after success we get our `bbl` file in the `riscv-pk/build` directory, which contains the boot loader, the linux kernel and the memory filesystem we made
```
make qemu
```

### 3.1.4 Running Linux BBL on QEMU

* Go back to the `QEMU-DASICS` directory and run the following command (assuming that the bbl path we got in the previous step is `$(PATH_TO_BBL)`)：
```
cd riscv64-softmmu
./qemu-system-riscv64 -machine virt -bios none -kernel $(PATH_TO_BBL) -m 1G -nographic -append "console=ttyS0 rw root=/dev/vda" -drive file=../img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0
```

* Wait for linux to boot until you get to the shell, the `/root` directory holds 4 compiled dasics tests, `/root/scripts/run-dasics-test.sh` is the script to run all the tests, execute the following command：
```
sh root/scripts/run-dasics-test.sh
```

* You can see that the output contains printouts of four tests, indicating that the dasics function is working correctly.

![](https://github.com/DASICS-ICT/.github/blob/main/qemu-linux.gif)

<h3 id=pynq ></h3>

## 3.2 NutShell-DASICS PYNQ-Z2 FPGA Boot Tutorial

This tutorial is a guide on how to put our modified NutShell processor core and Linux scare to run on the PYNQ-Z2 FPGA board.

Again, if you don't want to do the following steps, you can use the [dasics-pynq package] given in our release (https://github.com/DASICS-ICT/NutShell-DASICS/releases/download/nutshell-dasics-v1.0. 0/dasics-pynq.tar.gz), which contains the prepared BOOT.BIN and RV_BOOT.bin, and you can jump directly to [onboard steps](#fpga-onboard)

### 3.2.1 Preparation

* Install [mill](https://com-lihaoyi.github.io/mill/mill/Intro_to_Mill.html)

* Make sure python is in the $PATH

* Install [Vivado 2019.2](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)

* Download the PYNQ-Z2 Board File on the [PYNQ-Z2 page](https://www.tulembedded.com/FPGA/ProductsPYNQ-Z2.html) and add it to Vivado/2019.2/data/boards/ in the Vivado installation directory board_files in the Vivado installation directory

### 3.2.2 Make PYNQ Memory Image

This step is basically similar to the one in QEMU, so let's briefly describe the differences:

* You need to `export RISCV` variable and `RISCV_ROOTFS_HOME` variable first like in QEMU

* Run `make all` in `riscv-rootfs` directory

* Check `riscv-linux/arch/riscv/Kconfig`, `CONFIG_ZYNQ_ONBOARD` needs to be set to `y`

* Run `make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- zynq_dasics_defconfig` in `riscv-linux`

* Run `make` in riscv-pk. The generated bbl.bin is in the riscv-pk/build directory. Rename bbl.bin to RV_BOOT.bin.

### 3.2.3 Generate Verilog

* Get the NutShell-DASICS repository and go to the dev-dasics-ucas-os branch
```
git clone https://github.com/DASICS-ICT/NutShell-DASICS.git
cd NutShell-DASICS
git checkout dev-dasics-ucas-os
```
* Run the following command:
```
make BOARD=pynq
```

### 3.2.4 Generate Boot Image with Vivado Project

* Make sure vivado is in $PATH (may need source Vivado/2019.2/settings64.sh in the Vivado installation directory)
 
* Modify -jobs 20 on lines 2 and 6 of fpga/board/run.tcl to the appropriate concurrent number
 
* Get the [Xilinx/device-tree-xlnx](https://github.com/Xilinx/device-tree-xlnx) repository in the NutShell-DASICS directory, and switch to the xilinx-v2019.2 tag
 
* Run the following command to generate the boot image：
```
cd fpga && make PRJ=prj BOARD=pynq STANDALONE=true bootgen
```
* After successful operation, the fpga/boot/build/prj-pynq/BOOT.BIN file is generated

<h3 id=fpga-onboard ></h3>

### 3.2.5 Run on FPGA Board

* Prepare and partition the SD card, and format the first partition of it.

* Copy BOOT.BIN and RV_BOOT.bin to the first partition of the SD card

* After the board is powered on, you can see the boot information of linux kernel, the rest of the steps are the same as QEMU,
