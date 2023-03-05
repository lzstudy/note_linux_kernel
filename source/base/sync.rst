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
