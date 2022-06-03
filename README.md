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

zCore [yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc) 分支进展

- 经过 gdb 调试，修复了 `fork` 次数多会 `page fault` 的问题。主要原因有两个，其一是由于若一个线程与其他线程共享内存，在这个线程结束时，**会写 0**（根本原因）到 `clear_child_tid` 指针中（futex 相关）并唤醒一个在等待共享内存的线程。解决方法是在写之前检查地址的合法性，也尝试过刷新 TLB，未果，故还是用前一种方法；其二是在调度器的 `WEAK Executor` 部分，`return` 和 `drop` 的顺序有误，应该是先 `drop` 掉已经结束的任务再 `return`，否则可能在接下来的调度中执行已经结束的任务，造成 `page fault`
- 内核 `sleep` 时间不准确，发现是有的地方出现了相对时间和绝对时间进行比较的问题，已进行修复。鉴于这个问题对时间测量有比较大影响，应尽快向主仓库提 issue 或 pull request
- 已经可以在单或多核上通过 forktest，较为复杂的 kernel_intr_test（中断测试）和 coretest（`fork`、`sleep` 和输出的非周期混合）只能在单核上通过，上述自行设计的测例源码均在仓库的 ucore-user 文件夹下
- 对于 coretest（主要通过其测量性能），目前在多核上 `fork` 部分运行正常，但后续直接卡死在内核，需要进一步调试。除此之外，该测例目前在 qemu 上的时间测量很不准确，争取尽快在板子上进行尝试

调度器仓库 [yuzc](https://github.com/OSLab-zCore/PreemptiveScheduler/commits/yuzc) 分支进展

- 支持根据多核负载均衡执行新 `spawn` 任务
- 加入了简单的优先级更新调度机制，但目前仍有一些编译警告需要修复

*2022.5.2 updated*

*2022.5.3 modified*

*2022.5.6 modified*

zCore [yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc) 分支进展

- 解决之前时间统计为负的情况，主要原因是传入的参数与用户库中不符
- 对于 coretest（主要通过其测量性能），修复其目前在多核上直接卡死的问题（主要通过将多核调度器实装），目前用时基本和核数成反比
- 对于单核调度器（调度器仓库的主分支），如果 zCore 运行在两个（或更多）核上，coretest 卡死在 `fork` 后的输出，kernel_intr_test 的输出也不对
- 对于多核调度器，有一定概率出现借用报错（rust 相关），可能是某些锁的问题
- 整理上述问题的复现方法到文档中，发给张译仁助教，下一步是与他沟通调试

调度器仓库 [yuzc](https://github.com/OSLab-zCore/PreemptiveScheduler/commits/yuzc) 分支进展

- 解决编译警告，实装在上面的 zCore 分支上
- 支持动态输出在哪个核上跑的任务
- 基于优先级的调度需要讨论一下必要性，严格保序会出现数据结构锁的问题，影响性能，所以目前没有实装

*2022.5.9 updated*

*2022.5.12 modified*

*2022.5.13 modified*

zCore [yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc) & [lsm-yzc-merge](https://github.com/OSLab-zCore/zCore/tree/lsm-yzc-merge) 分支进展

- 对多核支持中修复的 bug 提交 PR 到主分支

- 提交测例状态修改的 PR 到 [zcore-test](https://github.com/rcore-os/zcore-tests) 仓库

- 多核多任务出现的死锁，定位到 kernel-sync 仓库的互斥锁问题

*2022.5.20 modified*

zCore [yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc) 分支进展 & 调度器仓库 [master](https://github.com/OSLab-zCore/PreemptiveScheduler/commits/master) 分支进展 & 调度器仓库 [yuzc](https://github.com/OSLab-zCore/PreemptiveScheduler/commits/yuzc) 分支进展

- 解决了两个比较关键的问题，其一是单核调度器跑多核 `sleep` 卡死，原因是时钟中断（本应 local）的处理并没有在每个核上都初始化，导致只有主核可以处理时钟中断，原先只允许在一个核上处理的原因不明（通过一个原子变量保证）；其二是多核调度器跑多核 shell 有概率没法进行输入，经调试发现，如果任务被分配到主核上，可以正常，如果在副核上，原因是只有主核能通过外设中断（在这里就是键盘输入）进入中断处理程序，全局唤醒等待中断的协程，而主核在没有任务时只会空转，不会开中断，副核虽然有 shell 这个任务，开中断，但没有办法处理，在宏观上造成无法输入的情况
- 总的进度是单核/多核调度器跑多核/单核 zCore 都没有问题，本地通过 coretest，多核调度器跑多核的 libc-test 等更新 CI 后在线跑
- kernel-sync 仓库死锁问题已由张译仁助教解决

板子上跑测例进展

- 编写打包测例程序，希望用一个程序直接在板子上跑完所有测例，不用进终端，能省去写驱动的时间（当然有时间还是会写）
- 目前 zCore 更新了编译方式，我在 Mac 上尝试编译 rootfs 会出问题（shadow-rs 的依赖被写死到只能支持 x86_64），目前尚不明确这样的目的

*2022.5.23 updated*

*2022.5.26 modified*

*2022.5.27 modified*

于子淳：

- 更新后的单核/多核调度器已经通过 CI
- 在集成测例的过程中，并撰写说明文档
  - 单核调度器：https://github.com/OSLab-zCore/zCore/actions/runs/2426925844


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

---------

刘松铭：完成SD卡的制作，研发出一套网络起zCore的操作。目前已经可以成功进入内核。

1. SD制作的文档记录在[制作SD卡流程](制作SD卡流程.md).
2. 网络起zCore的方法记录在[板子起zCore多核流程](板子起zCore多核流程.md).
3. 目前SD卡内的镜像并不是原厂镜像。我们尝试了制作原厂镜像但是失败。SD卡的镜像是Ubuntu官方给U740做的镜像，内部的Uboot可以使用。此外，目前只是能进入到内核，但是还不能完全起来，还有bug。

*2022.5.14 updated*

-------

刘松铭：BitmapAllocator分配失败的bug已经解决。目前的问题是U740给的可用地址不能都访问，会发生read page fault。估计可能是end的返回有问题。

*2022.5.20 updated*

------

刘松铭：zCore单核已经可以在U740上boot起来，多核还存在问题。

进展：

- 解决了PageFault的问题：将device tree占用的空间从free memory中剔除，并且增加一个对此的只读页表映射；可用物理的地址的最高为0xFFFFF000，避免上取整后溢出。
- 解决了找不到plic的问题：利用dtc将原始的dtb反汇编成可读文本，从中找到plic的tag，修改parse的代码，使得能找到plic。在有了plic后，u740就能处理外部中断。原本不需要plic，而现在需要，是因为多核起来存在核间中断。

当前问题：

- 多核起来后会在很奇怪的位置发生执行PageFault，地址在0x7FFFFFFE，原因暂时没找到。

*2022.5.27 updated*

--------

当前解决的bug

- 解决了找不到时钟中断handler的bug，原因是后面获取irq的逻辑有问题。但不知道为什么在Qemu上没有发现这样的问题

当前问题

- 单核能够正常启动，三核及以下能正常启动，但是到四个核会在`zcore_loader::linux::run()`处发生执行PageFault。

可选的操作

- 将页表扩展为16G
- 串口输入的支持

*2022.5.30 updated*

-----

刘松铭：zCore多核已经可以在u740上boot起来。

进展

- 最后发现是依赖库有问题。因为依赖库内写死了最多四个核，核的编号是`0,1,2,3`，但是u740的主核编号是`1,2,3,4`。
- 已经在本地打了patch，并向有问题的仓库提交了pr（包括[kernel-sync](https://github.com/DeathWish5/kernel-sync)和[PreemptiveScheduler](https://github.com/DeathWish5/PreemptiveScheduler)）。
- 在u740上测试了测例，全部通过。
- 将[u740分支](https://github.com/OSLab-zCore/zCore/tree/u740)更新上主线的进度。

下一步计划

- 串口输入的支持
- 测多核调度器的性能以及正确性（测例），与单核调度器对比，录制DEMO

*2022.6.3 updated*

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
