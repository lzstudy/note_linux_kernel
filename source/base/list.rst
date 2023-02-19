内核链表
=================

1. 双向链表
-----------

1.1 参考例子
************

.. code-block:: c

    #include <linux/list.h>

    struct fox {
        uint8_t weigth;
        uint8_t height;
        struct list_head list;
    };

    static struct fox new_fox;

    # 1 初始化链表头
    static LIST_HEAD(fox_list);

    # 2 初始化链表节点
    INIT_LIST_HEAD(&new_fox.list);

    # 3 插入节点
    list_add(&new_fox.list, &fox_list);                  // 在fox_list后插入
    list_add_tail(&new_fox.list, &fox_list);             // 在fox_list前插入

    # 4 删除节点, 注意删除节点后, 还需要手动释放内存
    list_del(list);

    # 5 移动和合并链表节点
    list_move(list, head);                              // 移除list, 并插入head链表后
    list_move_tail(list, head);                         // 移除list, 并插入head链表前

    # 6 判断链表是否为空
    list_empty(head);

    # 7 遍历链表 - 常用方案
    struct fox *f;
    list_for_each_entry(f, &fox_list, list) {
        // 对 f进行处理
    }

    # 8 遍历链表 - 其他方案
    list_for_each_entry_reverse(pos, head, member);     // 反向遍历
    list_for_each_entry_safe(pos, next, head, member);  // 如果遍历的同时要删除元素, 请使用此函数, 用list_for_each_entry会崩溃


.. note::

    list_entry(ptr, type, member)

    获取包含链表的结构体, list_entry本质上和container_of一样, 重新define了而已
    




