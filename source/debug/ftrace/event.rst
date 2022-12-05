tracer - event
==============

1 介绍
------

event分为静态和动态两种, 静态event也成为tracepoint, 动态event一般基于kprobe/uprobe.

2 静态event
-----------

========= ============
重要文件   说明
set_event 要追踪的事件
========= ============

2.1 流程
********

.. code:: c

   # 1 关闭trace
   echo 0 > tracing_on 

   # 2 设置要检测的事件
   echo v4l2_qbuf > set_event

   # 3 设置tracer
   echo nop > current_tracer 

   # 4 清空
   echo > trace

   # 5 打开
   echo 1 > tracing_on 

   # 6 报错数据
   cat trace_pipe > /tmp/trace.out &

2.2 查看支持的event
*******************

.. code:: c

   cat available_events

2.3 设置事件
************

.. code:: c

   echo v4l2_qbuf > set_event

3 动态事件
-----------

============== ===================================
重要文件        说明
kprobe_events  内核动态trace设置和查询
kprobe_profile 动态事件触发次数
uprobe_events  用户空间函数动态trace事件设置和查询
uprobe_profile 动态事件触发次数
============== ===================================

3.1 流程
********

.. code-block:: c

   # 1 关闭trace
   echo 0 > tracing_on 

   # 2 清空trace
   edho > trace

   # 3 设置tracer
   echo nop > current_tracer 

   # 4 设置动态点
   echo 'p:myprobe xxx_set_dma_addr base_addr=%x0 ds_ch=%x1 y_addr=%x2 cb_addr=%x'

   # 5 查看是否设置成功
   cat kprobe_events

   # 6 打开trace
   echo 1 > tracing_on

   # 7 使能event
   echo 1 > events/kprobes/enable
