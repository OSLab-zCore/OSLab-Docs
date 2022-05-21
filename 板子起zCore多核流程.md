# U740起zCore多核

本文档描述了在U740上起zCore多核的过程。

## 准备

将[zCore](https://github.com/rcore-os/zCore) clone到本地，参考[已有文档](./参考资料/doc.pdf)对其进行修改

以下是对原文档的一些修改：

1. 对编译image过程的一些修改

    首先将[u740的设备树文件](./资源文件/hifive-unmatched-a00.dtb)放到`zCore/target/riscv64/release`下。

    接着`zCore/target/riscv64/release`下新建its文件，用于打包FIT镜像

    ```c++
    /*
     * U-Boot uImage source file for "zCore-FU740"
     */
     
    /dts-v1/;
     
    / {
        description = "U-Boot uImage source file for zCore-FU740";
        #address-cells = <1>;
     
        images {
            kernel@zCoreFU740 {
                description = "Linux kernel for zCore-FU740";
                data = /incbin/("./zcore.bin.gz");
                type = "kernel";
                arch = "riscv";
                os = "linux";
                compression = "gzip";
                load = <0x80200000>;
                entry = <0x80200000>;
            };
            fdt@zCoreFU740 {
                description = "Flattened Device Tree blob for zCoreFU740";
                data = /incbin/("./hifive-unmatched-a00.dtb");
                type = "flat_dt";
                arch = "riscv";
                compression = "none";
            };
        };
     
        configurations {
            default = "conf@zCoreFU740";
            conf@zCoreFU740 {
                description = "Boot Linux kernel with FDT blob";
                kernel = "kernel@zCoreFU740";
                fdt = "fdt@zCoreFU740";
            };
        };
    };
    ```

    接着在[已有文档](./参考资料/doc.pdf)的基础上修改Makefile

    即，在最后一行加上`endif`，改一下`link_user_img`为`link-user-img`；下面还要多加一个make rule

    > 并添加对应编译feature：  
    >
    > ```makefile
    > # Makefile
    > ifeq ($(PLATFORM), fu740)
    > 	features += board_fu740 link-user-img
    > endif
    > # ...
    > fu740:
    > 	gzip -9 -cvf $(build_path)/zcore.bin > $(build_path)/zcore.bin.gz
    > 	mkimage -f $(build_path)/zcore-fu740.its $(build_path)/zcore-fu740.itb
    > 	@echo 'Build zcore fu740 FIT-uImage done'
    > ```

    注意：这时编译出来的是FIT镜像文件，而不是之前的uImage。

2. 目前不需要这个操作，SD卡的制作参考[这个文档](./制作SD卡流程.md)

   >在官网下载编译好的freedom-u-sdk，windows上可以用rufus将其装入SD卡中  

3. 最后一行加上一条命令

   >执行以下命令生成镜像 (在根目录下) 
   >
   >```bash
   >make riscv-image
   >cd zCore
   >make MODE=release LINUX=1 ARCH=riscv64 PLATFORM=fu740
   >make fu740 MODE=release LINUX=1 ARCH=riscv64 PLATFORM=fu740
   >```

4. 修改Cargo.toml

   在`zCore/Cargo.toml`中增加

   ```bash
   # Run on u740
   board_fu740 = []
   ```

5. 串口的波特率选择115200

## 网络起zCore

基本原理是通过一台主机搭建tftp服务，在板子上通过Uboot访问网络，从网络把主机中的镜像文件下载到内存，再从内存中boot。

首先在主机配置tftp服务，参考[CSDN上的教程](https://blog.csdn.net/weixin_45309916/article/details/109178659?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.control&spm=1001.2101.3001.4242)。

接着开启板子，按任意键打断Uboot，进行一些基础配置（第一次启动需要，后面不需要）

```bash
# 第一次需要设置环境变量
=> dhcp # 获取自己的ip, 注意此时可能会自动下载服务器中的镜像, 请通过Ctrl+C打断
=> setenv ipaddr 192.168.50.3 # 这是上一步获得的ip
=> setenv serverip 192.168.50.95 # 服务器的ip
=> ping 192.168.50.95 # 可以试一下ping主机
=> saveenv # 保存配置到flash
# 之后可以直接跳到这里开始
=> tftp 0xc2000000 zcore-fu740.itb # 下载镜像到内存
=> bootm 0xc2000000 # 启动
```

