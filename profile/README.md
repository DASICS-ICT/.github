### DASICS介绍
* DASICS(Dynamic in-Address-Space Isolation by Code Segments)是一种安全处理器设计方案，通过对不同代码片段访问的内存地址空间进行隔离并设置各自的访存权限，从而实现对非预期的越界访存和跳转的防护。这类越界访存可能来自于包括第三方恶意代码, 软件bug, 利用猜测执行的（如Spectre）漏洞在内的各种情况。

* 可以访问下载pdf版本的[用户手册](https://github.com/DASICS-ICT/DASICS-DOC)

### 现有仓库介绍
* 我们实现了对RISC-V架构Linux内核的修改，以支持DASICS相关安全处理机制，主要修改在如下几个仓库：
  * [riscv-linux](https://github.com/DASICS-ICT/riscv-linux)
  * [riscv-pk](https://github.com/DASICS-ICT/riscv-pk)
  * [riscv-rootfs](https://github.com/DASICS-ICT/riscv-rootfs)

* 我们实现了[QEMU的DASICS版本](https://github.com/DASICS-ICT/QEMU-DASICS)

* 我们在开源的RISC-V处理器[NutShell](http://https://github.com/OSCPU/NutShell)上实现了DASICS的硬件原型,并成功在FPGA上启动Linux并进行简单的安全测试
  * 支持DASICS功能的NutShell在[这里](https://github.com/DASICS-ICT/NutShell-DASICS)
