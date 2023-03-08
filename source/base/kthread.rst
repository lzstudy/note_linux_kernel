内核线程
==========

1 基本说明
-------------

我们在写程序中, 很少用内核线程, 通常使用工作队列workqueue(参考中断笔记第4节)

2 参考代码
-----------

.. code-block:: c


    struct task_struct *task;

    static int zw_kthread(void *arg)
    {
        return 0;
    }

    static int zw_init(void)
    {
        task = kthread_create(zw_kthread, NULL, "zwthr");
        CHECK_R(task == NULL, -1);

        return 0;
    }
