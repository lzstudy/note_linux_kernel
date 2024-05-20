中断编程
=================

1 基本介绍
-----------

2. 上半部
----------------

2.1 特性
***********

- 上半部只做严格限时工作, 例如中断应答或复位硬件, 上半部工作时中断会被关闭
- linux中的中断处理程序是不可重入的, 同一个中断处理程序绝对不会嵌套
- 当执行一个中断处理程序时, 内核处于中断上下文
- 中断上下文不可睡眠
- 中断处理程序拥有了自己的栈, 每个处理器一个, 大小为一页, 这个栈称为中断栈, 32位机一般为4K
- /proc/interrupts查看中断信息
- 锁提供保护机制, 防止其他处理器的并发访问
- 下半部: 软中断(固定32个, 内部已经分配好了), tasklet(常用), work queues, 内核定时器(准确的推迟)

2.2 参考代码
*************

.. code-block:: c

    #include <asm/system.h>
    #include <asm/irq.h>
    #include <linux/hardirq.h>

    # 1 申请中断
    request_irq()

    # 2 待标记禁止/激活当前处理器中断
    unsigned long flags;
    local_irq_save(flags);                          // 禁止中断
    local_irq_restore(flags);                       // 恢复中断

    # 3 禁止指定中断线
    disable_irq(irq);                               // 如果当前中断正在执行, 则阻塞等待执行完后, 禁止该中断
    disable_irq_nosync(irq);                        // 无需等待
    enable_irq(irq);                                // 使能中断, 上面函数可以嵌套, 调用多少次, 需要调用enable_irq多少次, 才能启用, 否则无效
    synchronize_irq(irq);                           // 等待一个特定的中断退出

    # 4 判断自己是否在中断上下文, 常用in_interrupt() == 0 来判断自己是否在进程上下文
    in_interrupt();                                 // 返回非0, 表示内核正在执行中断/下半部程序
    in_irq();                                       // 返回非0, 表示内核正在执行中断


    # 5 禁止/激活当前处理器中断(本地中断)
    # 一般该接口通过汇编实现, 另外此接口为无条件执行, 风险较大(不会记录之前的状态)
    local_irq_disable();
    local_irq_enable();

    
3 下半部 - tasklet
--------------------

3.1 特性
************

- tasklet是基于软中断实现的
- 两个不同的tasklet可以在不同处理器上执行, 相同类型的tasklet不能同时执行
- tasklet同一个处理程序多个实例不能再多个处理器上同时运行
- 多处理器, 通过TASKLET_STATE_RUN来判断是否被其他处理器执行
- tasklet不能睡眠
- tasklet和中断共享数据无须锁保护, tasklet和其他tasklet/软中断共享数据则需要锁保护
- 为了更好的利用高速缓存, tasklet总在调度他的处理器上运行
- 当内核中出现大量软中断时, 内核会唤醒一组线程来执行, 不过以最低优先级(nice 19)来运行, 每个处理器都会有一个该线程(ksoftirq/x)

3.2 参考代码
**************

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

4 下半部 - workqueue
-----------------------

4.1 特性
**************

- 在进程上下文执行, 既允许重新调度/睡眠
- workqueue和tasklet选择, 如果不需要睡眠就选tasklet, 需要就workqueue
- 默认的工作线程叫events/n, 我们一般都采用默认的工作队列(无须创建, 每个cpu带一个)

4.2 参考代码
**************

.. code-block:: c

    # 1 创建工作
    DECLARE_WORK(name, func, data);                 // 静态
    INIT_WORK(work, func, data);                    // 动态

    # 2 调度队列
    schedule_work(&work);
    schedule_delayed_work(&work, delay);            // 延时一段时间后执行

    # 3 刷新队列, 常用于卸载模块, 该函数会一直等待, 直到所有的对象都被执行完
    # 此函数会导致休眠, 所以只能在进程上下文中指向性
    flush_scheduled_work(void);

    # 4 取消workqueue
    cancel_delayed_work(&work);

.. note:: 
    
    - 多个工作队列同级FIFO模式, 优先级高先执行
    - 同一工作队列内部按时间先后执行

    




