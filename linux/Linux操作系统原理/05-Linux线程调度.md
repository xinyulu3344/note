## 线程属性

| 属性                | 属性值                                                       | 解释                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Detach state        | PTHREAD_CREATE_JOINABLE                                      |                                                              |
| Scope               | PTHREAD_SCOPE_SYSTEM(SCS)<br />PTHREAD_SCOPE_PROCESS(PCS)    | 线程竞争范围，Linux只支持SCS，也就是1:1模型                  |
| Inherit scheduler   | PTHREAD_INHERIT_SCHED                                        | 继承调度                                                     |
| Scheduling policy   | SCHED_OTHER<br />SCHED_IDLE<br />SCHED_BATCH<br />SCHED_FIFO<br />SCHED_RR | 调度策略。前三种调度策略是normal 调度策略；后两种是实时调度策略。normal调度策略的优先级必须设置为0。实时调度策略的优先级范围是[1, 99]，数字越大，优先级越高 |
| Scheduling priority | 0                                                            | 调度优先级                                                   |
| Guard size          | 4096 bytes                                                   |                                                              |
| Stack address       | 0x7f7b6fc78000                                               |                                                              |
| Stack size          | 0x800000 bytes                                               |                                                              |



