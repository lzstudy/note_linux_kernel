workqueue
============

1 特性
--------

- 在进程上下文执行, 既允许重新调度/睡眠
- workqueue和tasklet选择, 如果不需要睡眠就选tasklet, 需要就workqueue
- 默认的工作线程叫events/n, 我们一般都采用默认的工作队列(无须创建, 每个cpu带一个)

2 参考代码
------------

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