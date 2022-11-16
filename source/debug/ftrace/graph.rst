tracer - function_graph
=========================

1. 基本介绍
------------

graph tracer用于跟踪函数调用过程

2. 使用流程
------------

.. code-block:: c

    # 设置function tracer
    echo graph_function > current_tracer

    # 设置要跟踪的函数, 以uvc_probe为例
    echo "uvc_probe" > set_graph_function

    # 开启traceer
    echo 1 > tracing_on

    # 查看结果
    cat trace

.. image:: graph.jpg