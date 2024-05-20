elf加载过程
======================

1 基本说明
------------------

  1. fork阶段, 创建task
  2. exec阶段, 加载程序, 并跳转到libc库中
  3. 运行阶段, 执行到main函数

1.1 术语
******************

================= ==================== ====================
pt(pt_reg)        process trace         进程跟踪
sched             sheduling             调度
filp              file pointer          文件指针
================= ==================== ====================

2 各阶段说明
-----------------

2.1 fork阶段
*******************

  linux采用的是写实复制, 因此许多资源先用父进程的, 到需要修改时再进行修改

  1. 拷贝process
  2. 加入调度队列等待调度

.. code-block:: c

    do_fork()                [fork.c]
      copy_process()                    # 拷贝task
      wake_up_new_task()                # 唤醒task(将其加入调度队列)


2.2 exec阶段
*******************

  linux内核支持多种可执行文件, 程序种使用 ``struct linux_bin_prm`` 来描述可执行文件, 使用多种 ``struct linux_binfmt``
  来加载可执行文件。因此elf加载流程可总结为如下:

  1. 描述可执行文件类型, 初始化并读取elf头来设置 ``linux_bin_prm``
  2. 根据 ``linux_bin_prm`` 来找到对应的加载架构 ``linux_binfmt``, elf文件加载函数为 ``load_elf_binary``
  3. load_elf_binary将elf文件加载到内存, 并建立好内存映射
  4. 启动程序 ``start_thread``

======================= ====================== ====================
格式                     linux_binfmt定义       load_binary函数
a.out                    aout_format           load_aout_binary
elf                      elf_format            load_elf_binary
script脚本               script_format          load_script
em86                     em86_format            load_format
======================= ====================== ====================

.. code-block:: c

    do_execveat_common()
      prepare_binprm()                      # 准备binprm, 用于判断可执行文件的类型
      exec_binprm()
        search_binary_handler()
          load_elf_binary() [binfmt_elf.c]  # 加载elf二进制
            load_elf_phdrs()                # 读取elf头, 来判断该二进制文件是否为elf文件
              open_exec()                   # 如果文件含有动态库, 就需要打开动态库, 比如_main就属于动态库中的
              for:elf_map()                 # 建立物理内存与虚拟内存映射(页表), 但是没有实际将磁盘数据调入到内存
            create_elf_tables()
            start_thread()                  # 设置线程的入口, 一般是动态库的__libc_start_main, 然后进入

    ## 子流程 - elf_map
    elf_map()
      vm_mmap()
        do_mmap_pgoff() [mmap.c]


.. note:: 
    
    - 再建立虚拟内存时, 并没有将磁盘内容调入内存, 而是再读写时, 触发缺页异常, 然后
      将磁盘数据调入内存。此时触发的是硬件缺页异常(hard page fault)

    - start_thread当前的流程是再内核中, 而最终系统调用结束后要返回到应用层, 而应用层的返回点
      保存再pt_regs中, 因此修改此结构体数据, 就可以改变会到应用后的流程, 从而进入__libc_start_main中

    - pt_regs是进程从应用层到内核层时, 用户寄存器的状态。再系统调用陷入内核后要保存应用层的寄存器, 而
      当系统调用结束后, 再通过此结构恢复


2.3 调度阶段
*******************
