内核队列
=================

1. 标准队列
---------------

1.1 参考例子
*************


.. code-block:: c

    #include <linux/kfifo.h>

    static struct kfifo fifo;

    # 创建队列
    ret = kfifo_alloc(&fifo, PAGE_SIZE, GFP_KERNEL);
    CHECK_RET(ret);
    
    # 入队
    ret = kfifo_in(&fifo, buf, size);
    CHECK_RET(ret);

    # 出队
    ret = kfifo_out(&fifo, buf, size);
    CHECK_RET(ret);

    # 获取队列总大小
    size = kfifo_size(&fifo);

    # 获取剩余空间大小
    avial = kfifo_avail(&fifo);

    # 判断空/满, 0表示非空/满, 其他相反
    kfifo_is_empty(&fifo);
    kfifo_is_full(&fifo);

    # 清空队列
    kfifo_reset(&fifo);

    # 释放队列
    kfifo_free(&fifo);

    

    


    




