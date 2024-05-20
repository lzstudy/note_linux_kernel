内核探针
==========

1. kprobe
--------------

    kprobe可以再某个函数执行前和执行后加入处理, 直接修改参考程序简单修改即可

.. code-block:: c

    # 1 打开参考代码
    vi ${linux}/samples/kprobes/kprobe_example.c

    # 2 修改要调试的符号
    static char symbol[MAX_SYMBOL_LEN] = "isp_s_comp";

    # 3 添加执行前处理
    static int handler_pre(struct kprobe *p, struct pt_regs *regs)
    {
        pr_info("process is %s pid = %d\n", current->comm, current->pid);
        return 0;
    }

    # 4 添加执行后处理
    static void handler_post(struct kprobe *p, struct pt_regs *regs, unsigned long flags)
    {
    }



2. jprobe
-------------

    需要jprobe函数和检测函数相同


3. kretprobe
--------------

    kretprobe用于查看函数的返回值

.. code-block:: c

    # 1 打开参考带啊吗
    vi kernel/samples/kprobes/kretprobe_example.c

    # 2 修改要调试的符号
    static char func_name[NAME_MAX] = "_do_fork";

4. 配合调试技巧
----------------

4.1 查看应用层调用程序
***********************

.. code-block:: c

    # 在kprobe.c中添加如下内容
    #include <asm/current.h>
    pr_info("process is %s pid = %d\n", current->comm, current->pid);


