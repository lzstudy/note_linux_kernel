tracer - kprobe
===============

1 介绍
------

   使用该kprobe tracer可以代替简单使用jprobe查看参数, 和retprobe参看返回值。

2 使用流程
----------

.. code:: bash

   # 1 先关闭功能, 否则会提示device busy
   echo 0 > events/kprobes/enable

   # 2 设置kprobe zw0会生成目录, 所以不要重复, 否则报错, 建议使用函数名+k/r/j
   echo 'p:zw0 xxx' >> kprobe_events

   # 3 设置jprobe, 超过4个参数需要通过stack来保存
   echo 'p:zw1 xxx a=%r0 b=%r1 c=%r2 d=%r3 e=$stack0 f=$stack1' >> kprobe_events

   # 4 设置kretprobe, 
   echo 'r:zw2 xxx $retval' >> kprobe_events
   echo -:zw1 >> kprobe_events

   # 5 开启trace
   echo 1 > events/kprobes/enable

   # 6 触发事件

   # 7 查看结构
   cat trace

.. tip::

   * echo > kprobe_events 可以清空event

.. warning::

   * 设置jprobe时 a=%r0, b=%r1 等参数需要根据架构来设置, 可通过查看源码或者内核探针调试, 否则设置不正确会报 -22


3 zwtool使用流程
---------------------

.. code-block:: c

   # 设置kprobe, 常用于查找函数是否被执行到
   kprobe -kp func

   # 设置kprobe/jprobe/retprobe
   kprobe -kp -k/-j/-r func