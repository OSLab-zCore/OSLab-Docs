# Glommio

- Thread-per-core

  - 多Thread的问题
    - 锁昂贵：获取锁本身的开销+等待锁的时间
    - 切换上下文昂贵
  - 使用协程解决上述问题，就像函数调用，主动yield自己；一个core只有一个thread，没有切换开销；协程自身的异步性使得锁不再必要；把中断等系统调用分给专门的核（可选）

- 调度考虑的指标

  Tag（延迟敏感，吞吐率需求），延迟敏感的优先执行，有吞吐率需求的获取较长的时间片；shares：保证各个协程占用的总CPU时间与优先级成正比

- 三个ring

  latency ring：优先处理延迟敏感的task

  main ring：大多数任务的常驻ring

  poll ring：处理系统中断，攒成一堆统一向内核发请求