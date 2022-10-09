emmc检测
========

1 简单检测
----------

.. code:: c

   # 写入速度测试
   dd if=/dev/zero of=test bs=1M count=500 conv=fsynv oflag=direct
   500+0 records in
   500+0 records out
   524288--- bytes(524MB, 500MiB)copied, 6.23301 s, 84.1MB/s

   # 读取速度测试
   dd if=test of=/dev/zero bs=1M iflag=direct
   500+0 records in
   500+0 records out
   524288000 bytes(524 MB, 500 MiB)copied, 2.01331 s, 260MB/s
