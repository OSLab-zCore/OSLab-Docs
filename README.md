# OSLab-Docs

## Todos

### 第一阶段：修复zCore多线程

- 使得zCore通过functional下的多线程用例

  用例有：

  1. /libc-test/src/functional/pthread_cancel.exe 

     依赖的syscalls：

     `pthread_create`、`pthread_cancel`、`pthread_join`

  2. /libc-test/src/functional/pthread_cancel-points.exe 

  3. /libc-test/src/functional/pthread_cond.exe                    

  4. /libc-test/src/functional/pthread_mutex.exe                   

  5. /libc-test/src/functional/pthread_mutex_pi.exe                  

  6. /libc-test/src/functional/pthread_robust.exe                   

  7. /libc-test/src/functional/pthread_tsd.exe 

- 使得zCore通过regression下的多线程用例

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