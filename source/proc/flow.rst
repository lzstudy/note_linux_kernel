流程分析
==========

1 fork生成进程流程分析
--------------------------

.. code-block:: c

    # 相关文件 kernel/fork.c
    fork() => clone() => do_fork() => _do_fork() => copy_process()
                                                 => copy_sss()

2 exec流程分析
-----------------

3 exit流程分析
---------------

4 schedule()分析
-------------------

4.1 相关文件
**************

========================== ===============================================
文件                        说明
kernel/sched/stop_task.c   stop_sched_class
kernel/sched/deadline.c    dl_sched_class
kernel/sched/rt.c          rt_sched_class
kernel/sched/fair.c        fair_sched_class, 从红黑树选出vtime最小的进程
kernel/sched/idle_task.c   直接调度idle进程
========================== ===============================================

4.2 相关结构
**************

========================== ===============================================
数据结构                    说明
struct rq                  运行队列, 每个CPU都有一个
struct rq_rq               实时运行队列, 用于管理实时任务调度实体
struct sched_rt_entity     实时调度实体, 参与调度(调度单位不是线程, 是实体)
struct task_group          组调度实体
========================== ===============================================

4.3 相关函数
***************

================================== =======================================
函数                               说明
preempt_disable()                  禁止内核抢占
int smp_processor_id()             获取当前代码是在哪个处理器上运行的
================================== =======================================

4.4 调度流程
***************

.. code-block:: c

    ################ 调度流程 ################
    schedule() => __schedule(false) => pick_next_task() | context_switch()

    ############################################# 1 _schedule
    # 1.1 关闭本地中断
    local_irq_disable();
    # 1.2 记录切换次数
    switch_count = &prev->nivcsw;
    # 1.3 更新就绪队列时钟
    update_rq_clock(rq);
    # 1.4 挑选一个优先级最高的任务将其排进队列
    next = pick_next_task(rq, prev);
    # 1.5 清除pre的TIF_NEED_RESCHED标志
    clear_tsk_need_resched(prev);
    # 1.6 清除内核抢占标识
    clear_preempt_need_resched();
    # 1.7 上下文切换
    rq = context_switch(rq, prev, next);

    ############################################# 2 pick_next_task
    # 2.1 如果当前线程全部都是普通线程, 那么采用CFS调度
    struct task_struct *p = fair_sched_calss.pick_next_task(rq, prev);

    # 2.2 如果含有实时线程
    #     挑选规则会依次查询stop_sched_class、dl_sched_class、rt_sched_class、fair_sched_class、idle_sched_class
    #     选择最合的进程进行调度
    for_each_class(class)
       p = class->pick_next_task(rq, prev);

    ############################################# 3 context_switch
    # 3.1 挂起当前进程, 将其状态存在内存中

    # 3.2 获取要切换进程的上下文, 并恢复其状态

    # 3.3 跳到程序PC处, 恢复程序执行
    


