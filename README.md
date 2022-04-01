# OSLab-Docs

## Todos

### 第一阶段：修复zCore多线程

- 使得zCore通过functional下的多线程用例

  用例有：

  1. /libc-test/functional/pthread_cancel.exe 

     实现了`tkill`，`kill`，`tgkill` ，<font color=#7FFF00>**已通过**</font>，内容在lsm分支。

  2. /libc-test/functional/pthread_cancel-static.exe

     <font color=#7FFF00>**已通过**</font>，内容在lsm分支

  2. /libc-test/src/functional/pthread_cancel-points.exe 

     **时好时不好**

  4. /libc-test/functional/pthread_cancel-points-static.exe

     <font color=#7FFF00>**已通过**</font>，内容在lsm分支

  3. /libc-test/src/functional/pthread_cond.exe                    

  4. /libc-test/src/functional/pthread_mutex.exe                   

  5. /libc-test/src/functional/pthread_mutex_pi.exe                  

  6. /libc-test/src/functional/pthread_robust.exe                   

  7. /libc-test/src/functional/pthread_tsd.exe 

- 使得zCore通过regression下的多线程用例

  1. /libc-test/src/regression/pthread_cancel-sem_wait.exe

     **时好时不好**

  2. /libc-test/regression/pthread_cancel-sem_wait-static.exe

     <font color=#7FFF00>**已通过**</font>，内容在lsm分支。

  3. /libc-test/src/regression/pthread_exit-cancel.exe

     **有希望修好**

  4. /libc-test/regression/pthread_exit-cancel-static.exe 

     **有希望修好**
  
  5. /libc-test/regression/pthread_exit-dtor.exe
  
     <font color=#7FFF00>已通过</font>，内容在lsm分支。
  
  6. /libc-test/regression/pthread_exit-dtor-static.exe 
  
     <font color=#7FFF00>**已通过**</font>，内容在lsm分支。
  

### 第二阶段：调度器设计

- waker page 保序和锁的竞争做一个权衡：需要测试用户调度频率、zCore 协程执行时间等
- weak executor 应该放在池子中而不是释放
- 区分用户任务和纯内核任务：把用户程序当成线程，延迟敏感单独放 Ring，加入 tag 等
- 只要时钟中断就 yield 不科学，因为不确定本任务执行了多久
- 多核，专核专用，动态调整，一个核一个 runtime，利用 shared memory 通信
- 编写测试用例

## 调研汇总
- glommio

     链接：https://www.datadoghq.com/blog/engineering/introducing-glommio/
     
     描述：
     
- naive fifo executor

     链接：https://github.com/rcore-os/executor
     
     描述：就是一个fifo的executor
     
- PreemptiveScheduler

  链接：https://github.com/Deathwish5/PreemptiveScheduler
  
  描述：
  
- tokio

  链接：https://github.com/tokio-rs/tokio
  
  描述：
  
- monoio

  链接：https://github.com/bytedance/monoio
  
  描述：

- smol
  
  链接：https://github.com/smol-rs/async-executor
  
  描述：代码比较精简，https://zhuanlan.zhihu.com/p/137353103