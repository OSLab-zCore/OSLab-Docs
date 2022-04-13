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
  7. /libc-test/functional/pthread_tsd.exe
  8. /libc-test/functional/pthread_tsd-static.exe
  9. /libc-test/regression/pthread-robust-detach.exe
  10. /libc-test/regression/pthread-robust-detach-static.exe
  11. /libc-test/regression/pthread_cond-smasher.exe
  12. /libc-test/regression/pthread_cond-smasher-static.exe
  13. /libc-test/regression/pthread_condattr_setclock.exe
  14. /libc-test/regression/pthread_condattr_setclock-static.exe
  15. /libc-test/regression/pthread_once-deadlock.exe
  16. /libc-test/regression/pthread_once-deadlock-static.exe
  17. /libc-test/regression/pthread_rwlock-ebusy.exe
  18. /libc-test/regression/pthread_rwlock-ebusy-static.exe
  19. /libc-test/src/regression/pthread_cancel-sem_wait.exe
  20. /libc-test/regression/pthread_cancel-sem_wait-static.exe
  21. /libc-test/src/regression/pthread_exit-cancel.exe
  22. /libc-test/regression/pthread_exit-cancel-static.exe
  23. /libc-test/regression/pthread_exit-dtor.exe
  24. /libc-test/regression/pthread_exit-dtor-static.exe
- 可以进一步完善的地方：
  1. async的`handle_signal()`
  2. 完善tkill、tgkill、kill等的参数支持
  3. 修改一些为了Debug编写的不优雅代码，避免给后人挖坑

----------------

中期报告结束，中期报告的ppt和部分讲稿在[OSLab-Docs](https://github.com/OSLab-zCore/OSLab-Docs)仓库（这里的仓库均指本organization的仓库）下的`中期报告/`下。刘松铭编写的zCore信号支持在[zCore](https://github.com/OSLab-zCore/zCore)仓库下的[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支，于子淳编写的futex相关支持在[yuzc](https://github.com/OSLab-zCore/zCore/tree/yuzc)分支，二人合并的代码在[lsm-yzc-merge](https://github.com/OSLab-zCore/zCore/tree/lsm-yzc-merge)分支。

*2022.4.5 updated*

----------

刘松铭新增任务：在rcore-tutorial-v3上实现一个简化的signal机制，能让app的signal handler处理来自其他app或内核的signals。参考链接：https://github.com/rcore-os/rCore-Tutorial-v3/blob/ch9/os/src/task/signal.rs

*2022.4.10 updated*

---------

[lsm](https://github.com/OSLab-zCore/zCore/tree/lsm)分支的修改进度

- 去除了`handle_signal()`的hard codes
- 去除了tkill、tgkill、kill等的hard codes，完善了部分的参数支持
- 去除了不必要的注释，在没有完全实现linux标准的位置加上了`warn!`，避免给后人挖坑

Todos

- 完善对x86的支持
  - kernel-hal/src/common/context.rs
  - linux-object/src/signal/mod.rs
- 修复外部库`rcore-fs`的bug，并去除测试时使用的hard codes
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

*2022.4.13 modified* 

-----

新发现一个 x86 无法通过但 riscv可以通过的测例：/libc-test/src/regression/pthread_cancel-sem_wait.exe

*2022.4.13 updated* 


### 第二阶段：调度器设计

- waker page 保序和锁的竞争做一个权衡：需要测试用户调度频率、zCore 协程执行时间等
- weak executor 应该放在池子中而不是释放
- 区分用户任务和纯内核任务：把用户程序当成线程，延迟敏感单独放 Ring，加入 tag 等
- 只要时钟中断就 yield 不科学，因为不确定本任务执行了多久
- 多核，专核专用，动态调整，一个核一个 runtime，利用 shared memory 通信
- 编写测试用例

---

于子淳新增任务：

- 先解决抢占式调度器通过不了测例的问题

- 使用 `virt` 模式（较弱）开两个核心，测试不同程序调度：
  - `sleep`
  - `fork` 子进程
  - 隔一秒输出一次

- 仿照 `Linux` 的三层调度器来实现

- 调高时钟中断频率

- `uring` 分支要看懂

*2022.4.11 updated*

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

