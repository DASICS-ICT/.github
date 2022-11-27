### DASICS介绍
* DASICS(Dynamic in-Address-Space Isolation by Code Segments)是一种安全处理器设计方案，通过对不同代码片段访问的内存地址空间进行隔离并设置各自的访存权限，从而实现对非预期的越界访存和跳转的防护。这类越界访存可能来自于包括第三方恶意代码, 软件bug, 利用猜测执行的（如Spectre）漏洞在内的各种情况。

* 可以访问用户[手册仓库](https://gitee.com/dasics/dasics-doc)，或者访问[网页版用户手册](https://roxanneucas2016.gitee.io/dasics_blog/2022/01/13/dasics-manual/)查看DASICS的使用说明


### 现有仓库介绍
* 我们实现了对RISC-V架构Linux内核的修改，

* 我们实现了QEMU的DASICS版本，

* 我们实现了在[NutShell](http://https://github.com/OSCPU/NutShell)上的硬件原型,并成功在FPGA上启动Linux并进行简单的安全测试。对应仓库可以在[这里](https://gitee.com/dasics/nut-shell-dasics)找到


### 联系
邮箱: TODO（dasics mail list）
