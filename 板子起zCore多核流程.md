# U740起zCore多核

将[zCore](https://github.com/rcore-os/zCore) clone到本地，参考[已有文档](./参考资料/doc.pdf)对其进行修改

以下是对原文档的一些修改：

1. 在最后一行加上`endif`，改一下`link_user_img`为`link-user-img`，下面还要多加一个make rule

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
    > 	mkimage -A riscv -O linux -C gzip -T kernel -a 80200000 -e 80200000 -n "zCore-fu740" -d $(build_path)/zcore.bin.gz $(build_path)/zcore-fu740
    > 	@echo 'Build zcore fu740 image done'
    > ```

2. 目前不需要这个软件

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

4. 多一步

   在`zCore/Cargo.toml`中增加

   ```bash
   # Run on u740
   board_fu740 = []
   ```

5. 波特率选择115200