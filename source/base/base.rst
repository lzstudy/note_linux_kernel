基本接口
=========

1 内存申请
-----------

.. code-block:: c

    #include <linux/slab.h>

    kmalloc(size, GFP_KERNEL);


2 延时接口
------------

.. code-block:: c

    #include <linux/delay.h>

    void ndelay(unsigned long nsecs); 
    void udelay(unsigned long usecs); 
    void mdelay(unsigned long msecs);



