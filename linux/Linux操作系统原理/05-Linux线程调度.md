## 线程属性

| 属性                | 属性值                                                       | 解释                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Detach state        | PTHREAD_CREATE_JOINABLE                                      |                                                              |
| Scope               | PTHREAD_SCOPE_SYSTEM(SCS)<br />PTHREAD_SCOPE_PROCESS(PCS)    | 线程竞争范围，Linux只支持SCS，也就是1:1模型                  |
| Inherit scheduler   | PTHREAD_INHERIT_SCHED                                        | 继承调度                                                     |
| Scheduling policy   | SCHED_OTHER<br />SCHED_IDLE<br />SCHED_BATCH<br />SCHED_FIFO<br />SCHED_RR | 调度策略。前三种调度策略是normal 调度策略；后两种是实时调度策略。normal调度策略的优先级必须设置为0。实时调度策略的priority value范围是[1, 99]，数字越大，优先级越高 |
| Scheduling priority | 0                                                            | 调度优先级                                                   |
| Guard size          | 4096 bytes                                                   |                                                              |
| Stack address       | 0x7f7b6fc78000                                               |                                                              |
| Stack size          | 0x800000 bytes                                               |                                                              |

对于normal调度策略：PR = 20 + NI，NI∈[-20,19]，PR∈[0, 39]

对于real time调度策略：PR = -1 - priority value，priority value∈[1,99]。PR∈[-100, -2]

PR越大，优先级越低

## 查看优先级

```bash
# PR和NI -> 仅对normal有效
top -p pid
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
   10 root      rt   0       0      0      0 S   0.0  0.0   0:00.21 watchdog/0


# 查看进程调度策略和priority value
chrt -p pid
pid 10's current scheduling policy: SCHED_FIFO
pid 10's current scheduling priority: 99
```

## 修改优先级

```bash
# 设置进程调度策略和priority value
chrt -f -p PRIORITY PID

Policy options:
 -b, --batch          set policy to SCHED_BATCH
 -d, --deadline       set policy to SCHED_DEADLINE
 -f, --fifo           set policy to SCHED_FIFO
 -i, --idle           set policy to SCHED_IDLE
 -o, --other          set policy to SCHED_OTHER
 -r, --rr             set policy to SCHED_RR (default)
```

## 调度策略的过程

对于Normal进程，比如SCHED_OTHER调度策略，使用分时调度，调度器是CFS，动态优先级由NICE值控制，NICE越小，优先级越高

对于real-time进程，SCHED_FIFO调度策略，PR小的先执行，在PR相同时，先入先出。SCHED_RR调度策略，PR小的先执行，在PR相同时，轮询

## 代码

输出线程属性

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <pthread.h>
#include <time.h>
#include <stdlib.h>
#include <errno.h>
#define _GNU_SOURCE

#define handle_error_en(en, msg) \
               do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void display_pthread_attr(pthread_attr_t *attr, char *prefix){
    int s, i;
    size_t v;
    void *stkaddr;
    struct  sched_param sp;

    s = pthread_attr_getdetachstate(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getdetachstate");

    printf("%sDetach state        = %s\n", prefix,
            (i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :
            (i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :
            "???");

    s = pthread_attr_getscope(attr, &i);
        if (s != 0)
        handle_error_en(s, "pthread_attr_getscope");
        printf("%sScope               = %s\n", prefix,
            (i == PTHREAD_SCOPE_SYSTEM)  ? "PTHREAD_SCOPE_SYSTEM" :
            (i == PTHREAD_SCOPE_PROCESS) ? "PTHREAD_SCOPE_PROCESS" :
            "???");

    s = pthread_attr_getinheritsched(attr, &i);
        if (s != 0)
        handle_error_en(s, "pthread_attr_getinheritsched");
        printf("%sInherit scheduler   = %s\n", prefix,
            (i == PTHREAD_INHERIT_SCHED)  ? "PTHREAD_INHERIT_SCHED" :
            (i == PTHREAD_EXPLICIT_SCHED) ? "PTHREAD_EXPLICIT_SCHED" :
            "???");

    s = pthread_attr_getschedpolicy(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getschedpolicy");
    printf("%sScheduling policy   = %s\n", prefix,
        (i == SCHED_OTHER) ? "SCHED_OTHER" :
        (i == SCHED_FIFO)  ? "SCHED_FIFO" :
        (i == SCHED_RR)    ? "SCHED_RR" :
        "???");

    s = pthread_attr_getschedparam(attr, &sp);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getschedparam");
    printf("%sScheduling priority = %d\n", prefix, sp.sched_priority);

    s = pthread_attr_getguardsize(attr, &v);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getguardsize");
    printf("%sGuard size          = %zu bytes\n", prefix, v);

    s = pthread_attr_getstack(attr, &stkaddr, &v);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getstack");
    printf("%sStack address       = %p\n", prefix, stkaddr);
    printf("%sStack size          = 0x%zx bytes\n", prefix, v);

}

void* computing(void* arg){
    int s;
    pthread_attr_t gattr;

    s = pthread_getattr_np(pthread_self(), &gattr);
    if (s != 0)
        handle_error_en(s, "pthread_getattr_np");

    printf("Thread attributes:\n");
    display_pthread_attr(&gattr, "\t");

    pause();

    pthread_exit(0);

}


int main(){

    pthread_t tid;
    pthread_attr_t attr;

    pthread_attr_init(&attr);

    // 线程创建函数
    pthread_create(&tid, &attr, computing, NULL);

    // 等待指定的线程结束
    pthread_join(tid,NULL);

    return 0;
}
```

输出结果：

```
Thread attributes:
        Detach state        = PTHREAD_CREATE_JOINABLE
        Scope               = PTHREAD_SCOPE_SYSTEM
        Inherit scheduler   = PTHREAD_INHERIT_SCHED
        Scheduling policy   = SCHED_OTHER
        Scheduling priority = 0
        Guard size          = 4096 bytes
        Stack address       = 0x7f2f00670000
        Stack size          = 0x800000 bytes

```

