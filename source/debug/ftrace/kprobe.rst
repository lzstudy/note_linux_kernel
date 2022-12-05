tracer - kprobe
===============

1 介绍
------

使用该tracer可以代替kprobe、jprobe、kretprobe

2 使用流程
----------

.. code:: bash

   # 1 先关闭功能, 否则会提示device busy
   echo 0 > events/kprobes/enable

   # 2 设置kprobe/jprobe/kretprobe
   echo 'p:zw0 xxx' > kprobe_events
   echo 'p:zw1 xxx a=%r0 b=%r1 c=%r2 d=%r3 e=$stack0 f=1$stack' >> kprobe_events
   echo 'r:zw2 xxx $retval' >> kprobe_events
   echo -:zw1 >> kprobe_events

   # 3 开启trace
   echo 1 > events/kprobes/enable

   # 4 触发事件

   # 5 查看结构
   cat trace

.. tip::

   * echo > kprobe_events 可以清空event

.. warning::

   * 设置jprobe时 a=%r0, b=%r1 等参数需要根据架构来设置, 可通过查看源码或者内核探针调试, 否则设置不正确会报 -22