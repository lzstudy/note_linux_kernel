编译内核模块
============

设置多文件编译
--------------

::

   obj-m += xxx.o                           # 设置要编译的模块
   xxx-objs += aaa.o bbb.o                  # 设置该模块依赖的文件

设置编译选项
------------

::
   
   ccflags-y += -std=c11                    # 设置c11标准
   ccflags-y += -I$(PWD)/../common          # 设置编译的头文件
