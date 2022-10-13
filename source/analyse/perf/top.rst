perf功能
========

1. top, record, report
----------------------

.. code:: c

   # 使用场景, 分析进程号为1234
   perf top -g -p 1234

1.1 top
*******

实时显示占用CPU时钟最多的函数或指令, 可以用来查找热点函数

.. code:: c

   perf top

   # 以突变方式显示加起来为100%
   perf top --call-graph fractal

   # 以图表方式显示
   perf top --call-graph graph

1.2 record和report
******************

record保存数据, 保存后的数据要用perf report解析

.. code:: c

   perf record
   perf report

2. stat
-------

.. code:: c

   perf stat










