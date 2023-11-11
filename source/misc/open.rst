open
====

1. 介绍
-------

系统调用是一种特殊的异常，通常归于同步异。它是通过SVC指令触发的，异常处理程序可以
从异常状态寄存器(ESR)中得到SVC指令使用的立即数。定义系统用 ``SYSCALL_DEFINEX``, 其中
X表示的是参数个数...

1.1 系统调用宏展开过程
**********************

.. code:: shell

   #define SYSCALL_DEFINEX(name, ...)   SYSCALL_DEFINEX(1, _##name, __VA_ARG__)
      #define SYSCALL_DEFINEX(x, sname, ...)   __SYSCALL_DEFINEX(x, sname, __VA_ARGS__)
         #define __SYSCALL_DEFINEX(x, name, ...)
            asmlinkage long sys##name(__SC_DECL##x(__VA_ARGS__)); ...

2 调用流程
----------

.. code::

   # 应用层
   open(tty) -> sys_open -> SyS_open

   # 内核层
   kernel_ventry 1, sync                                [中断向量表 第三部分sync, entry.S]
     el0_sync -> el0_svc -> el0_svc_handler             [entry.S]
	    el0_svc_common                                  [syscall.c]
          invoke_syscall -> do_sys_open -> do_filp_open [open.c]
		    path_openat -> vfs_open -> do_dentry_open   [open.c]
			  chrdev_open -> filp->f_op->open           [char_dev.c]
			    tty_open                                [tty_io.c]

   # 驱动层
   tty_open -> tty_init_dev -> driver->ops->install     [tty_io.c]
     uart_install                                       [tty_core.c]
	   tty_init_termios(tty) -> tty_termios_baud_rate   [tty_io.c]
	 tty_ldisc_setup -> n_tty_open -> n_tty_set_termios
	 uart_open                                          [serial_core.c]
	   tty_port_open -> uart_port_activate              [serial_core.c]
	     uart_startup -> imx_uart_startup               [imx.c]
		  imx_uart_dma_init -> dma_request_slave_channel[imx.c]




