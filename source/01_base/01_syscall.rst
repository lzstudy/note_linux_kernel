系统调用
================

1 基本说明
--------------------

    1. 每个系统调用都有唯一的系统调用号
    2. 所有的系统调用都返回long类型
    3. 用户空间的程序无法直接运行内核代码, 因为内核空间的地址在保护内存中
    4. 系统调用是基于软中断实现的, 触发异常然后进入内核, 入口为system_call
    5. 为了保证应用传入数据安全, 使用copy_from_user()来接收应用数据, 此接口可能会阻塞
    6. 内核在执行系统调用时, current指针有效, 指向当前任务
    7. 所有系统调用必须定义于<asm/unistd.h>中

1.2 宏说明
***************

====================== ======================================
SYSCALL_DEFINE0()      此宏定义的系统调用没有参数
====================== ======================================


2 系统调用流程
-----------------------

2.1 open基本流程
*****************

.. code-block:: c

    ################################### 1 中断部分
    vector_swi
     ret_fast_syscall          [entry-common.S]            # 进入中断
      SyS_open
       do_sys_open()           [open.c]                    # open系统调用
        getname()              [namei.c]                   # 获取名字
         kmem_cache_alloc()                                # 通过slab获取
          do_filp_open()

    ################################### 2 do_filp_open
    # do_filp_open 用于返回新的file对象
    do_filp_open()
     path_openat()
      get_empty_filp()        [file_table.c]              # 从slab获取一个未使用的file结构
       kmem_cache_zalloc(filp_cachep, GFP_KERNEL);
      path_init()                                         # 检测路径合法性
      do_last()                                           # 剩余处理
       lookup_open()
        vfs_create()                                      # 由于添加了create标记, 所以会创建

    ################################### 3 vfs_create
    vfs_create()
     ext3_create()              [namei.c]
      ext3_new_inode()
       ext3_alloc_inode()
        kmem_cache_alloc()
      ext3_mark_inode_dirty()






        


3 实现系统调用
--------------------

.. code-block:: c

    # 1 在系统调用表最后加一个表项
    # arm最新版本在arch/arm/kernel/calls.S 中定义

    # 2 系统调用号必须顶, usr/include/asm/unistd_32.h
    # 添加自己的系统调用号

    # 3 具体函数实现, 函数名要和1中的表项一样


