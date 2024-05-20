内核定时器
=============

1 基本特性
-------------

- 系统定时器默认值为100(既10ms)
- jiffies记录启动以来产生的节拍总数, 且用volatile标识
- 定时器以软中断在下半部执行

.. note:: 

    为了提供定时器搜索效率, 次啊用按超时时间分组(5组), 当前超时在某个组的阈值范围内, 就
    归入该组, 从而提高搜索效率。 
    
    - 所有定时器如果以链表形式存放, 内核会遍历整个链表, 开销很大
    - 链表以超时时间排序, 插入和删除的开销会很大

2 定时方案 - 忙等待
--------------------

.. code-block:: c

    # 1 完成占用CPU, 适合时间极短的情况, 可以再中断中使用
    unsigned long timeout = jiffies + 10;
    while(time_before(jiffies, timeout));

    # 2 可以让CPU投入其他工作, 适合时间较长的情况, 注意此方案不能再中断中使用
    unsigned long timeout = jiffies + 5 * HZ;
    while(time_before(jiffies, delay)), cond_resched());

3 短延时
---------

.. code-block:: c

    #include <linux/delay.h>

    udelay(us);
    ndelay(ns);
    mdelay(ms);

4 指定延时
-----------

.. code-block:: c

    # 比如等待队列上的任务既等待一个特定事件, 又等待一个特定时间
    # 因此将其设置为可中断状态, 然后用schedule_timeout()来代替schedule()
    set_current_state(TASK_INTERRUPTIBLE);
    schedule_timeout(5*HZ);

5 内核定时器
--------------

6 高精度定时器hrtimer
----------------------
