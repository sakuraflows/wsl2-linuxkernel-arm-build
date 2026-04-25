#     wsl2-linuxkernel-arm-build can running Waydorid
#### WSL2（ARM）编译可运行Waydorid的WSL2（ARM）内核
构建环境：
CPU：高通8cX Gen3
编译系统环境：Windows 11 on Arm 25H2，WSL2 Ubuntu 24.04 
注意：本流程为原生ARM下的非交叉编译流程。 

编译内核的版本：`WSL2-Linux-Kernel-linux-msft-wsl-6.18.20.1` 
架构：ARM64 
### 构建流程
##### 1 准备
配置好基础环境：
克隆此地址下的仓库：https://github.com/microsoft/WSL2-Linux-Kernel

根据微软官方的指导，安装如下依赖：
`$ sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio qemu-utils`

调整相关配置: 
编译配置文件的路径是：`WSL2-Linux-Kernel-linux-msft-wsl-6.18.20.1/Microsoft/config-wsl-arm64`,因为我要后期在WSL上运行waydorid（waydorid可以让linux运行安卓程序，但运行waydorid必须要对内核进行配置，这也是我为什么要自己编译内核和模块的原因）

修改配置及修改后的结果如下：
```
CONFIG_ANDROID_BINDER_IPC=y
CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"
```

##### 2 构建与编译
执行编译指令：
`cd WSL2-Linux-Kernel-linux-msft-wsl-6.18.20.1 # your source code path` 
`make ARCH=arm64 KCONFIG_CONFIG=Microsoft/config-wsl-arm64 -j$(nproc) && make ARCH=arm64 INSTALL_MOD_PATH="$PWD/modules" modules_install`

在编译过程中，可能会遇到如下提示：
```
* Restart config...
*
*
* Android
*
Android Binder IPC Driver (ANDROID_BINDER_IPC) [Y/n/?] y
Android Binderfs filesystem (ANDROID_BINDERFS) [N/y/?] (NEW)
```
建议选“y”

大约一小时后会编译完成。

在编译完成后，可以根据需要利用源代码中提供的脚本将模块打包为VHDX文件： 
`sudo ./Microsoft/scripts/gen_modules_vhdx.sh "$PWD/modules" $(make -s kernelrelease) modules.vhdx`


##### 3 执行清理：
`make clean` 
如果已经确定使用vhdx来加载模块，则可以执行官方指导中的`$ make clean & rm -r “$PWD/modules”`,否则只执行make clean即可。


### 简单的使用方法
以下方法比较简单，但我使用的是其他方法，所以不保证此部分的可行性  
确保 WSL2 使用自定义内核和模块，方法是修改 .wslconfig 文件（或使用 WSL 设置）。

```
[wsl2]
kernel=<your kernel path>
kernelModules=< your vhdx path>
```
详见：[ Waydroid in WSL2 with sound](https://gist.github.com/onomatopellan/c5220c0efddaff69aaff77cca80b7b8e)



### 参考：
- [Win11上配置Linux子系统+wsl-vscode](https://zhuanlan.zhihu.com/p/693938916) 
- [025 完美避坑版：Windows 11 原生运行安卓 (Waydroid) 完整教程](https://gdfr.dpdns.org/waydroid-guide/) 
- [Waydroid on WSL2](https://elkeid-me.github.io/posts/waydroid-on-wsl2) 
- [ Waydroid in WSL2 with sound](https://gist.github.com/onomatopellan/c5220c0efddaff69aaff77cca80b7b8e) 
