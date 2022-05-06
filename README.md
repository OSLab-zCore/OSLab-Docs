# OSLab-Docs

2022春OS大实验文档        刘松铭、于子淳

## 进度日志

### 第一阶段：修复zCore多线程

- 已经通过的多线程用例

  1. /libc-test/functional/pthread_cancel.exe
  2. /libc-test/functional/pthread_cancel-static.exe
  3. /libc-test/src/functional/pthread_cancel-points.exe
  4. /libc-test/functional/pthread_cancel-points-static.exe
  5. /libc-test/functional/pthread_cond.exe
  6. /libc-test/functional/pthread_cond-static.exe
  7. /libc-test/functional/pthread_robust.exe
  8. /libc-test/functional/pthread_robust-static.exe
  9. /libc-test/functional/pthread_tsd.exe
  10. /libc-test/functional/pthread_tsd-static.exe
  11. /libc-test/regression/pthread-robust-detach.exe
  12. /libc-test/regression/pthread-robust-detach-static.exe
  13. /libc-test/regression/pthread_cond-smasher.exe
  14. /libc-test/regression/pthread_cond-smasher-static.exe
  15. /libc-test/regression/pthread_condattr_setclock.exe
  16. /libc-test/regression/pthread_condattr_setclock-static.exe
  17. /libc-test/regression/pthread_once-deadlock.exe
  18. /libc-test/regression/pthread_once-deadlock-static.exe
  19. /libc-test/regression/pthread_rwlock-ebusy.exe
  20. /libc-test/regression/pthread_rwlock-ebusy-static.exe
  21. /libc-test/src/regression/pthread_cancel-sem_wait.exe
  22. /libc-test/regression/pthread_cancel-sem_wait-static.exe
  23. /libc-test/src/regression/pthread_exit-cancel.exe
  24. /libc-test/regression/pthread_exit-cancel-static.exe
  25. /libc-test/regression/pthread_exit-dtor.exe
  26. /libc-test/regression/pthread_exit-dtor-static.exe
- 可以进一步完善的地方：
  1. async的`handle_signal()`
  2. 完善tkill、tgkill、kill等的参数支持
  3. 修改一些为了Debug编写的不优雅代码，避免给后人挖坑

----------------

中期报告结束，中期报告的ppt和部分讲稿在[OSLab-Docs](https://github.com/OSLab-zCore/OSLab-Docs)仓库（这里的仓库均指本organization的仓库）下的`中期报告/`下。刘松铭编写的zCore信号支持在[zCore](https://github.com/OSLab-zCore/zCore)仓库下的[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支，于子淳编写的futex相关支持在[yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc)分支，二人合并的代码在[lsm-yzc-merge](https://github.com/OSLab-zCore/zCore/tree/lsm-yzc-merge)分支。

*2022.4.5 updated*

----------

刘松铭新增任务：在rcore-tutorial-v3上实现一个简化的signal机制，能让app的signal handler处理来自其他app或内核的signals。参考链接：https://github.com/rcore-os/rCore-Tutorial-v3/blob/ch9/os/src/task/signal.rs

需要实现的系统调用或结构如下：

- [x] sigaction（结构和系统调用）

  增加sigaction的结构，用于存储每个signal对应的action，action包含处理signal时应该设置的mask以及handler的地址。

- [x] kill（补全发射信号）

  将发射信号补全至31个（不包含第0个，第0个是默认信号）。通过kill系统调用传递信号。

- [x] sigprocmask

  新增mask结构体，mask的作用是block某些信号（用bitmap存储），被block的信号暂时不被处理。

- [x] sigreturn

  sigreturn时除去信号处理的标记，并且恢复trap frame。

- [x] 信号处理的过程

  仅处理当前不被block的信号。信号分为内核信号和用户信号，内核信号按照规定动作进行处理。此外，sigaction不允许为内核信号设置action；用户信号则根据sigaction进行处理，即保持当前的trap frame，将sepc改成handler的地址，mask设置为sigaction中对应信号的mask，将调用sigreturn的汇编指令插入栈中，将ra设为栈中汇编指令的开始地址。

  新增killed和frozen标志，对应SIGKILL和SIGSTOP，前者的作用是标记进程是否被杀死，后者是的作用是标记是否被冻结。被冻结的进程当前暂时先不处理信号，yield出去。下次被schedule的时候检查是否有信号，然后继续yield。如此往复直到被解冻或杀死。

  *更新：去除向栈中插入汇编指令的操作。因为这样会引起页错误（执行不可执行的代码），改为标准实现，即有用户库调用Sigreturn*

- [x] 用户库

  将所需的sigaction结构以及sys_call在用户库进行包装。

  测例将xv6的`sig_tests.c`重写为rust的测例。

*2022.4.10 updated*

*2022.4.18 modified* 

*2022.4.20 modified*

*2022.4.22 modified* 

---------

[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支的修改进度

- 去除了`handle_signal()`的hard codes
- 去除了tkill、tgkill、kill等的hard codes，完善了部分的参数支持
- 去除了不必要的注释，在没有完全实现linux标准的位置加上了`warn!`，避免给后人挖坑

Todos

- 完善对x86的支持
  - kernel-hal/src/common/context.rs
  - linux-object/src/signal/mod.rs
- 测试时使用的hard codes
  - linux-object/src/fs/mod.rs
  - linux-syscall/src/file/dir.rs
  - linux-syscall/src/file/fd.rs
- 修复SignalStack与musl数据结构不对齐的问题
  - linux-object/src/thread.rs
  - 原有测例CI不过：/libc-test/regression/sigaltstack.exe

[lsm-yzc-merge](https://github.com/OSLab-zCore/zCore/tree/lsm-yzc-merge) 分支的修改进度

- 去除了 futex、get_robust_list、set_robust_list 等的 hard codes
- 增加了对于函数参数的文档注释，在没有完全按照标准实现的位置加上了 `warn!`，避免给后人挖坑

Todos

- 测试 futex 相关测例在 x86 上的支持情况，预期改动不大

*2022.4.11 updated*

- 新发现一个 x86 无法通过但 riscv可以通过的测例：/libc-test/src/regression/pthread_cancel-sem_wait.exe


*2022.4.13 updated* 

-----

[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支的修改进度

- 已完善对x86的部分支持，支持修改ra
- 修复SignalStack数据不对齐的问题
- x86下可以通过/libc-test/src/regression/pthread_cancel-sem_wait.exe

todos

- /libc-test/regression/sigaltstack.exe测例的问题是一个新功能，设计signal换栈

*2022.4.15 updated*

[lsm-yzc-merge](https://github.com/OSLab-zCore/zCore/tree/lsm-yzc-merge) 分支的修改进度

- 修复 CI 的 Build 和 Deploy docs 相关问题
- Test CI zircon 的 libos 测例运行不稳定，助教和工程师建议多跑几次
- 修改测试文件，提交新增多线程相关测例的 pull request 到 [zcore-test](https://github.com/rcore-os/zcore-tests) 仓库

*2022.4.15 updated*

- 修复 CI 的 Build 和 Deploy docs 由原[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支的代码带来的问题

*2022.4.17 updated*

- 按标准实现了 `LOCK_PI` 和 `UNLOCK_PI`，修复了 /libc-test/functional/pthread_robust.exe 无法通过的问题
- 尝试 merge 到 master，但发现部分新增测例在合并后无法通过
- 修复了合并后 /libc-test/src/functional/pthread_cancel.exe 超时的问题
- 提交对多线程相关测例状态修改的 pull request 到 [zcore-test](https://github.com/rcore-os/zcore-tests) 仓库

*2022.4.19 updated*

- 提交 pull request 到 [zCore](https://github.com/rcore-os/zCore) 主仓库
- 修正 github url 的问题

*2022.4.20 updated*

- 通过 CI 和 reviewer 的检查，成功合并进 [zCore](https://github.com/rcore-os/zCore) 主分支 :fireworks:

*2022.4.21 updated*

### 第二阶段：调度器设计

- waker page 保序和锁的竞争做一个权衡：需要测试用户调度频率、zCore 协程执行时间等
- weak executor 应该放在池子中而不是释放
- 区分用户任务和纯内核任务：把用户程序当成线程，延迟敏感单独放 Ring，加入 tag 等
- 只要时钟中断就 yield 不科学，因为不确定本任务执行了多久
- 多核，专核专用，动态调整，一个核一个 runtime，利用 shared memory 通信
- 编写测试用例

-------

于子淳新增任务：

- [x] 先解决抢占式调度器通过不了测例的问题，通过的 CI 在 [zcore-update-lock](https://github.com/DeathWish5/zCore/commits/zcore-update-lock) 分支
- [x]  `uring` 分支代码阅读
- [x]  使用 `virt` 模式开多个核心（目前启动有问题），调高时钟中断频率，测试不同程序调度：
  - `sleep`
  - `fork` 子进程
  - 隔一秒输出一次
- [x]  riscv64-linux-musl-gcc 编译报错：在 Ubuntu 上重装
- [x]  "Modern Concurrency Platforms Require Modern System-Call Techniques" 论文阅读
- [ ]  仿照 `Linux` 的三层调度器来实现：待定

*2022.4.15 updated*

*2022.4.20 modified*

*2022.4.22 modified*

*2022.4.24 modified*

*2022.4.26 modified*

zCore [yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc) 分支进展

- 测试完成目前 zCore 多核正常的启动和执行（不包括复杂任务调度）

- 修改 rcore-user CMakeLists（去掉 `-march=rv64imac -mabi=lp64`，否则会需要 zCore 不支持的 `ld-musl-riscv64-sf.so.1`），解决了基于该用户库编写的用户测例在 zCore 上无法执行的问题

- 将编写的性能测例（注意 `wait` 的阻塞问题）加入 zCore，新的问题是 `fork` 次数多会造成 `page fault`

调度器仓库 [yuzc](https://github.com/OSLab-zCore/PreemptiveScheduler/commits/yuzc) 分支进展

- 增加调度器的多核任务窃取


*2022.4.26 updated*

*2022.4.27 modified*

*2022.4.29 modified*

----

刘松铭：跟随前人的工作在U740起zCore多核，但目前还有bug，猜测bug是由zCore主线更新引起的。原zCore起来的版本是一年前的版本。目前做了一些向新版本的适配，还在debug中。进度记录在[板子起zCore多核流程](./板子起zCore多核流程.md)。

*2022.4.29 updated*

-------

刘松铭：参考[官方文档](https://sifive.cdn.prismic.io/sifive/b9376339-5d60-45c9-8280-58fd0557c2f0_hifive-unmatched-gsg-v1p4_ZH.pdf)制作启动SD卡。

*2022.5.1 updated*

--------

刘松铭：利用假期几天时间尝试了几种制作SD卡的办法，但是都没有成功。进度记录在[制作SD卡流程](./制作SD卡流程.md)。目前问题还是无法恢复SD卡之前的数据，接下来会考虑如下的解决方案：

1. 尝试石振兴老师提供的链接，看看能不能恢复原来的数据。
2. 使用大容量的移动硬盘，继续按照官方文档制作镜像。
3. 想办法找个好的SD卡，拷贝数据。

*2022.5.7 updated*

## 调研汇总

- glommio

     链接：https://github.com/DataDog/glommio
     
     描述：基于io_uring的rust异步库，https://www.datadoghq.com/blog/engineering/introducing-glommio/
     
- naive fifo executor

     链接：https://github.com/rcore-os/executor
     
     描述：简单fifo调度器
     
- PreemptiveScheduler

  链接：https://github.com/Deathwish5/PreemptiveScheduler
  
  描述：张译仁学长基于王辉宇学长的工作改进的抢占调度器
  
- tokio

  链接：https://github.com/tokio-rs/tokio
  
  描述：异步rust库
  
- monoio

  链接：https://github.com/bytedance/monoio
  
  描述：字节实现的rust runtime

- smol
  
  链接：https://github.com/smol-rs/async-executor
  
  描述：代码比较精简，https://zhuanlan.zhihu.com/p/137353103
