基本接口
=========

1 模块模板
**************

===================== ============================
sample_module.tar.gz_ 简单的内核模块
===================== ============================

.. _sample_module.tar.gz: http://120.48.82.24:9300/driver/sample_module.tar.gz

2 内存申请
-----------

.. code-block:: c

    #include <linux/slab.h>

    kmalloc(size, GFP_KERNEL);


3 延时接口
------------

.. code-block:: c

    #include <linux/delay.h>

    void ndelay(unsigned long nsecs); 
    void udelay(unsigned long usecs); 
    void mdelay(unsigned long msecs);



