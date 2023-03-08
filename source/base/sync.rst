内核同步
============

1 原子操作
------------

1.1 原子整数操作
*******************

- 单纯的对整数或者位进行处理, 如果只是某个整形变量是竞争的, 那么可以采用此机制
- 原子操作通常是内联函数, 一般通过内嵌汇编实现
- 原子操作会比锁等开销小, 对高速缓存的影响也比较小

.. code-block:: c

    #include <asm/atomic.h>

    # 1 定义原子变量
    atomic_t u = ATOMIC_INIT(0);

    # 2 设置新值
    atomic_set(&u, 4);
    int val = atomic_read(&u);
    LOG_I("val = %d", val);

    # 3 加法
    atomic_add(2, &u);              // 加上某个数据
    atomic_inc(&u);                 // 自增

    # 4 减法
    atomic_sub(2, &u);              // 减去某个数据
    atomic_dec(&u);                 // 自减

    # 5 读取某个变量
    atomic_read(&u);

    # 6 判断
    atomic_add_and_test(&u);        // 原地加, 如果为0, 返回1
    atomic_dec_and_test(&u);        // 原地减, 如果为0, 返回1
    
.. note:: 
    
    上面是32位的原子操作, 64位只需在后面加上64即可, 例如atomic64_add

1.2 原子位操作
*******************

.. code-block:: c

    #include <asm/bitops.h>

    unsigned long word = 0;         // 必须是unsigned long类型

    # 1 置位
    set_bit(0, &word);              // 置位第0位
    set_bit(1, &word);              // 置位第1位

    # 2 清位
    clear_bit(1, &word);            // 清空第1位

    # 3 翻转位
    change_bit(3, &word);           // 翻转第3位

    # 4 测试位, 置位返回1
    test_bit(3, &word);             // 测试第3位

    # 5 搜索被设置的位
    find_first_bit(&word, 32);
    find_first_zero_bit(&word, 32);

.. note:: 
    
    对于位操作, 内核还提供了非原子操作方法, 例如__set_bit(), 
    主要用于平时我们的位域使用(挺有用的)


2 锁操作
------------

2.1 自旋锁
*************

- 短时间内进行轻量级加锁
- 自旋锁不能递归, 会造成死锁

.. code-block:: c

    #include <linux/rwlock.h>

    spinlock_t lock;
    unsigned long flags;

    # 初始化spinlock
    spin_lock_init(&lock);                           // 动态初始化

    # 方案一 带禁用本地中断的锁(常用)
    spin_lock_irqsave(&lock, flags);
    spin_unlock_irqrestore(&lock, flags);

    # 方案二 不带禁用中断的锁
    spin_lock(&lock);
    // 临界区
    spin_unlock(&lock)


.. warning::

    - 自旋锁用在中断处理程序时, 首先要禁止本地中断(防止中断多重进入导致类型递归的效果, 从而死锁)
    - 递归使用时, 程序不会崩, 会卡在那里

.. note:: 

    自旋锁的原理: 

    SMP系统:

       核内锁住调度器, 从而禁止在本CPU上的线程调度, 核与核之间自旋

    单核系统:

        锁住调度器, 此时spin_lock = preempt_disable
    

2.2 读写锁
*************

- 写操作完全互斥
- 读锁可以递归使用, 写锁不可以, 所以中断时可以用read_lock, 不过写就需要write_lock_irqsave

.. code-block:: c

    #include <linux/rwlock.h>

    rwlock_t lock;

    # 1 初始化
    rwlock_init(&lock);

    # 2 读临界区
    read_lock(&lock);
    // 临界区
    read_unlock(&lock);

    # 3 写临界区
    write_lock(&lock);
    // 临界区
    write_unlock(&lock);


2.3 顺序锁
*************

- 适合数据存在很多读者
- 你的写的数据很少, 但是优先于读, 且不允许读者让写者饥饿

.. code-block:: c

    #include <linux/seqlock.h>

    seqlock_t lock;

    # 1 初始化
    seqcount_init(&lock);

    # 2 写锁过程
    write_seqlock(&lock);
    jiffies_64 += 1;
    write_sequnlock(&lock);

    # 3 读锁过程
    do {
        seq = read_seqbegin(&lock);
        ret = jiffies_64;
    }while(read_seqretry(&lock, seq));

2.4 禁止抢占
--------------

.. code-block:: c

    # 增加抢占计数值, 从而禁止内核抢占
    preempt_disable();

    # 减少抢占计数, 当该值为0时, 检查和执行被挂起的需要调度的任务
    preempt_enable();

    # 激活抢占, 但不检查任何被挂起的调度任务
    preempt_no_resched();

    # 返回抢占计数
    preempt_count();


.. note:: 
    
    - 此功能会比自旋锁轻量, 以简洁的方式解决每个处理器上的数据访问问题

3 信号量
------------

3.1 信号量
*************

- 比自旋锁有更大的开销

.. code-block:: c

    #include <linux/semaphore.h>

    int ret;
    struct semaphore sem;

    # 1 初始化, 初始化值为1, 第一次获取就会获取到
    sema_init(&sem, 1);
    
    # 2 获取信号量, 并且进入可中断睡眠中, 为了ctrl c能用
    ret = down_interruptible(&sem);
    CK_RET(ret < 0, ret>);

    # 3 释放信号量
    up(&sem);


3.2 互斥锁
***********

.. code-block:: c

    #include <linux/mutex.h>

    struct mutex lock;

    # 1 初始化
    mutex_init(&lock);

    # 2 上锁
    mutex_lock(&lock);

    # 3 解锁
    mutex_unlock(&lock);


3.2 读写信号量
******************

.. code-block:: c

    #include <linux/rwsem.h>

    struct rw_semaphore sem;

    # 1 初始化
    init_rwsem(&sem);

    # 2 读信号量 - 获取/释放
    down_read(&sem);
    up_read(&sem);

    # 3 写信号量 - 获取/释放
    down_write(&sem);
    up_write(&sem);

.. note:: 
    
    - 读信号量可以连续的down_read(), 不会死锁
    - 写信号量不可以连续的down_write(), 否则会导致死锁, 内核会直接卡死在这里
    - 读写操作一般分开, down_read()信号量后, up_write信号量, 再down_write也会阻塞, 等待down_write

3.3 完成量
******************

- 常用于等待一个事件, 会经常使用

.. code-block:: c

    #include <linux/completion.h>

    struct completion done;

    # 1 初始化
    init_completion(&done);

    # 2 等待
    wait_for_completion(&done);

    # 3 发送信号表示完成
    complete(&done);

4 内存屏障
-----------

.. code-block:: c

    # 读屏障
    rmb()

    # 写屏障
    wmb()

    # 读写屏障
    mb()

5 等待队列
--------------

.. code-block:: c

    #include<linux/wait.h>

    struct wait_queue_head wq;

    # 1 初始化
    init_waitqueue_head(&wq);

    # 2 阻塞某个线程
    wait_event(&wq, -1);

    # 3 唤醒wait
    wake_up(&wq);

.. tip:: 
    
    - wake_up() 唤醒可中断 + 不可中断任务
    - wakeup_interruptible() 只唤醒可中断任务

.. note:: 
    
    wake_up()每次只能唤醒一个进程, 且是从队列头开始唤醒, wait_event()函数, 每次会
    将新建的等待队列插入到对头, 因此最后调用的wait_event最先唤醒. 可以使用wake_up_all()
    唤醒所有的线程, 然后通过condtion来判断是否是自己 
