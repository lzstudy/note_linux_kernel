中断基础
==========

1 中断特性
-----------------

- 上半部只做严格限时工作, 例如中断应答或复位硬件, 上半部工作时中断会被关闭
- linux中的中断处理程序是不可重入的, 同一个中断处理程序绝对不会嵌套
- 当执行一个中断处理程序时, 内核处于中断上下文
- 中断上下文不可睡眠
- 中断处理程序拥有了自己的栈, 每个处理器一个, 大小为一页, 这个栈称为中断栈, 32位机一般为4K
- /proc/interrupts查看中断信息
- 锁提供保护机制, 防止其他处理器的并发访问
- 下半部: 软中断(固定32个, 内部已经分配好了), tasklet(常用), work queues, 内核定时器(准确的推迟)

2 中断程序
-----------

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





