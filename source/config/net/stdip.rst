固定ip地址
==========

1. 适用于NXP_8MP平台
--------------------

.. code:: c

   vi /etc/systemd/network/10-eth.network

   [Match]
   Name=eth0
   KernelCommandLine=!root=/dev/nfs
   [Network]
   Address=192.168.0.232/24
   Gateway=192.168.0.1
   DNS=192.168.0.1
