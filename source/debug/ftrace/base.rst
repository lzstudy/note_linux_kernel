ftrace基础
===========

1. 基本介绍
-----------

ftrace是内核trace子系统的一种, ftrace下又细分为
function tracre、graph tracer、event tracer等. 
ftrace工作在虚拟文件系统之上(debugfs/tracefs)

1.1 学习路线
*************

1.2 参考书籍
*************

1.3 常用术语
************

2 功能说明
---------------

2.1 可挂载的文件系统
***********************

.. code-block:: c

    # 挂载到debugfs下
    mount -t debugfs debugfs /sys/kernel/debug

    # 挂载到tracefs下
    mount -t tracefs tracefs /sys/kernel/tracing

3 使用流程
-----------

.. code-block:: c

    # 1、挂载tracefs文件系统
    mount -t tracefs tracefs /sys/kernel/tracing

    # 2、关闭trace
    echo 0 > tracing_on

    # 3、设置当前要使用的tracer(这里以函数tracer为例)
    echo function_graph > current_tracer

    # 4、设置要跟踪的函数(以do_fork、init_module为例, 可以设置多个)
    echo do_fork > set_ftrace_filter
    echo inix_module > set_ftrace_filter

    # 5、设置函数调用深度
    echo 128 > max_graph_depth

    # 6、开启trace
    echo 1 > tracing_on

    # 7、查看trace
    cat trace          // 静态显示trace, 显示完buffer还在, echo > trace后清空buffer
    cat trace_pipe     // 动态显示trace信息
    snapshot           // 获取当前trace buffer备份
    trace_marker       // 把用户层的log插入trace buffer, 实现用户和内核trace log同步

.. note:: 
    
    cat trace_pipe > /tmp/trace.out & 输出日志到文件

4 常用技巧
-----------

4.1 被调式进程运行时间短或者不知道进程号, 可使用脚本检测
***********************************************************

.. code-block:: 
    
    sh -c “echo $$ > set_ftrace_pid; echo 1 > tracing_on; kill xxx; echo 0 > tracing_on”

4.2 如果函数名很多, 且相似, 可以使用正则匹配
***************************************************

.. code-block:: 
    
    echo ‘dev_attr_*’ > set_ftrace_filter

4.3 过滤模块函数
******************

.. code-block:: shell

    # 规则
    <function>:<command>:<parameter>

    # 例如过滤ext3模块的write-函数
    echo 'write*:mod:ext3' > set_ftrace_filter

4.4 从过滤列表删除某个函数
******************************

.. code-block:: 

    echo '!ip_rcv' >> set_ftrace_filter





5 其他说明
-----------

trace-cmd可以简化命令, 生成trace.data文件提供给 kernelShark 等UI工具解析, 实现trace的可视化

trace-cmd下载git clone [https://github.com/rostedt/trace-cmd.git](https://github.com/rostedt/trace-cmd.git)

使用 ./trace-cmd record -e hbpvt -e sched -e irq

# 参考网站 http://t.zoukankan.com/sky-heaven-p-5321553.html

https://zhuanlan.zhihu.com/p/479833554

