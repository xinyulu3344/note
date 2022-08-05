## 问题描述

系统日志中有如下报错：

```
Jul 27 23:58:06 mysql-008-009 kernel: INFO: task md10_raid1:20093 blocked for more than 120 seconds.
Jul 27 23:58:06 mysql-008-009 kernel: "echo 0 > /proc/sys/kernel/hung_task_timeout_secs" disables this message.
Jul 27 23:58:06 mysql-008-009 kernel: md10_raid1 D ffff9df93e245140 0 20093 2 0x00000080
Jul 27 23:58:06 mysql-008-009 kernel: Call Trace:
```

服务器存储用的是4块盘做的软raid10。

## 代码分析

上面的日志，是在`linux/kernel/hung_task.c`内核文件，`check_hung_task(struct task_struct *t, unsigned long timeout)`函数中调用打印的。

hung task的核心思想是创建一个内核监测进程循环监测处于D状态的每一个进程（任务），统计它们在两次检测之间的调度次数，如果发现有任务在两次监测之间没有发生任何的调度则可判断该进程一直处于D状态，很有可能已经死锁，因此触发报警日志打印，输出进程的基本信息，栈回溯以及寄存器保存信息以供内核开发人员定位。

在内核的subsys初始化阶段就会调用hung_task_init()函数，调用kthread_run()函数创建了一个名为khungtaskd的内核线程，立即尝试调度执行watchdog()函数。该线程就是专用于检测D状态进程的后台内核线程。

```c
static int __init hung_task_init(void)
{
    atomic_notifier_chain_register(&panic_notifier_list, &panic_block);
    watchdog_task = kthread_run(watchdog, NULL, "khungtaskd");

    return 0;
}
```

在watchdog中，首先设置优先级为0，即一般优先级，不影响其它进程。然后进入主循环，每隔timeout时间执行一次。`timeout` =`sysctl_hung_task_timeout_secs`=`CONFIG_DEFAULT_HUNG_TASK_TIMEOUT`，`CONFIG_DEFAULT_HUNG_TASK_TIMEOUT`可以由内核配置选项修改：`kernel.hung_task_timeout_secs = 120`，默认是120s。120s睡眠结束被唤醒，执行`check_hung_uninterruptible_tasks()`监测函数。如果设置为0，则该值会被内核设置为长整型long的最大值，相当于永远不会监测。

```c
/*
 * kthread which checks for tasks stuck in D state
 */
static int watchdog(void *dummy)
{
    set_user_nice(current, 0);

    for ( ; ; ) {
        unsigned long timeout = sysctl_hung_task_timeout_secs;

        while (schedule_timeout_interruptible(timeout_jiffies(timeout)))
            timeout = sysctl_hung_task_timeout_secs;

        check_hung_uninterruptible_tasks(timeout);
    }

    return 0;
}
```

检查内核是否已经DIE或者panic，如果是，则表明内核已经crash，直接返回。如果不是，则进入监测流程，对内核中每一个线程进程检查。这个过程是在RCU加锁状态下进行，所以不可能检查所有的线程，否则在进程较多的情况下会导致加锁时间太长。这里通过max_count和batch_count来限制检查的数量。max_count由sysctl_hung_task_check_count指定，默认为PID最大个数PID_MAX_LIMIT，batch_count由HUNG_TASK_BATCHING指定，默认为1024。相当于每批次检查1024个线程。

轮询线程的过程中，只对UNINTERRUPTIBLE状态的进程进行检查，调用check_hung_task函数，入参为task_struct结构和超时时间(120s)

```c
/*
 * Check whether a TASK_UNINTERRUPTIBLE does not get woken up for
 * a really long time (120 seconds). If that happens, print out
 * a warning.
 */
static void check_hung_uninterruptible_tasks(unsigned long timeout)
{
    int max_count = sysctl_hung_task_check_count;
    int batch_count = HUNG_TASK_BATCHING;
    struct task_struct *g, *t;

    /*
     * If the system crashed already then all bets are off,
     * do not report extra hung tasks:
     */
    if (test_taint(TAINT_DIE) || did_panic)
        return;

    rcu_read_lock();
    do_each_thread(g, t) {
        if (!max_count--)
            goto unlock;
        if (!--batch_count) {
            batch_count = HUNG_TASK_BATCHING;
            if (!rcu_lock_break(g, t))
                goto unlock;
        }
        /* use "==" to skip the TASK_KILLABLE tasks waiting on NFS */
        if (t->state == TASK_UNINTERRUPTIBLE)
            check_hung_task(t, timeout);
    } while_each_thread(g, t);
 unlock:
    rcu_read_unlock();
}
```

首先，计算线程从创建至今总的调度次数。t->nvcsw表示主动放弃CPU的次数，t->nivcsw表示被抢占CPU的次数。unlikely函数意思是执行该分支的可能性小。因为流水线的作用，cpu在一条指令执行时，会同时完成下一条指令的取指，提高CPU的利用率。但是如果跳转到别的分支，那么预取的指令就没用了，这里用unlikely，表明执行这个分支的可能性不大。在编译成二进制代码时，会将else的语句往前放，增加程序执行速度。

这里判断线程是否被冻结，如果被冻结，则跳过检查。如果线程还没有被调度，即switch_count=0，则跳过检查。接下来判断上一次检查的switch_count和本次switch_count是否一样，如果不一样，说明本轮timeout(120s)内线程有被调度，则更新switch_count并返回，否则说明timeout时间内，线程一直都没有被调度，一直处于D状态。接下来判断报警次数有没有达到上限sysctl_hung_task_warnings，默认是10，可以通过内核配置参数kernel.hung_task_warnings修改。超过该值则直接返回，不再打印报警信息。也就是该进程如果一直处于D状态，那么默认会每2分钟打印一次告警，打印10次后不再打印。

接下来通过printk在控制台和日志中打印任务的名称、PID号、超时时间、内核tainted信息、sysinfo、内核栈barktrace以及寄存器信息等。如果开启了debug lock则打印锁占用的情况，并touch_nmi_watchdog以防止nmi_watchdog超时。

最后如果设置了sysctl_hung_task_panic标识则直接触发panic，可以通过内核配置参数kernel.hung_task_panic 进行配置，默认是0关闭。

```c
static void check_hung_task(struct task_struct *t, unsigned long timeout)
{
    unsigned long switch_count = t->nvcsw + t->nivcsw;

    /*
     * Ensure the task is not frozen.
     * Also, skip vfork and any other user process that freezer should skip.
     */
    if (unlikely(t->flags & (PF_FROZEN | PF_FREEZER_SKIP)))
        return;

    /*
     * When a freshly created task is scheduled once, changes its state to
     * TASK_UNINTERRUPTIBLE without having ever been switched out once, it
     * musn't be checked.
     */
    if (unlikely(!switch_count))
        return;

    if (switch_count != t->last_switch_count) {
        t->last_switch_count = switch_count;
        return;
    }
    if (!sysctl_hung_task_warnings)
        return;
    sysctl_hung_task_warnings--;

    /*
     * Ok, the task did not get scheduled for more than 2 minutes,
     * complain:
     */
    printk(KERN_ERR "INFO: task %s:%d blocked for more than "
            "%ld seconds.\n", t->comm, t->pid, timeout);
    printk(KERN_ERR "\"echo 0 > /proc/sys/kernel/hung_task_timeout_secs\""
            " disables this message.\n");
    sched_show_task(t);
    debug_show_held_locks(t);

    touch_nmi_watchdog();

    if (sysctl_hung_task_panic) {
        trigger_all_cpu_backtrace();
        panic("hung_task: blocked tasks");
    }
}
```

## 总结

```bash
# 打印hung告警的次数
kernel.hung_task_warnings = 10
# 监测线程检查线程是否处于D状态的时间间隔。如果设置为0，则相当于不进行监测。
kernel.hung_task_timeout_secs = 120
# 是否在出现120s D状态进程的情况下，直接panic。可以结合kdump搜集到vmcore
kernel.hung_task_panic = 0
```
