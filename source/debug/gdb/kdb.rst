kdb
===

1 配置bootargs
--------------

.. code:: c

   # 在uboot中bootargs追加如下内容
   setenv bootargs xxx kgdboc=ttymxc1,115200
   saveenv

2 触发kdb
---------

2.1 内核启动时
**************

.. code:: c

   # 开机后按下Pause Break键(不是ESC，笔记本键盘可能会没有), 内核启动10s左右会停住
   [0]kdb >

2.2 程序起来后配置
******************

.. code:: c

   echo "ttymxc1,115200" > /sys/module/kgdboc/parameters/kgdboc
   echo g > /proc/sysrg-trigger

