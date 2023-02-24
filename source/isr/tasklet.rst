tasklet
===========

1 特性
-------

- tasklet是基于软中断实现的
- 两个不同的tasklet可以在不同处理器上执行, 相同类型的tasklet不能同时执行
- tasklet同一个处理程序多个实例不能再多个处理器上同时运行
- 多处理器, 通过TASKLET_STATE_RUN来判断是否被其他处理器执行
- tasklet不能睡眠
- tasklet和中断共享数据无须锁保护, tasklet和其他tasklet/软中断共享数据则需要锁保护
- 为了更好的利用高速缓存, tasklet总在调度他的处理器上运行
- 当内核中出现大量软中断时, 内核会唤醒一组线程来执行, 不过以最低优先级(nice 19)来运行, 每个处理器都会有一个该线程(ksoftirq/x)

2 参考代码
-----------

.. code-block:: c

    #include <linux/interrupt.h>

    # 1 声明一个tasklet类
    DECLARE_TASKLET(name, func, data)

    # 2 调度tasklet
    tasklet_schedule(&my_tasklet);

    # 3 禁止下半部
    # 一般是先获取锁(SMP), 然后再禁止下半部(防止死锁)
    # 同样禁止内部有计数, 既禁止多少次, 需要使能多少次才能解开
    local_bh_disable();                     // 禁止本地软中断和tasklet
    local_bh_enable();                      // 激活本地软中断和tasklet


