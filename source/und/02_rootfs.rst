根文件系统挂载
=====================


1 基本参数
----------------

.. code-block:: c

    __read_mostly编译器属性, 通知内存此变量主要用于读取, 很少进行写入, 被此修饰的
    数据将自动被放到cache中, 以提高整个系统的执行效率


2 流程
---------------

========================================= ====================================
struct kmem_cache \*names_cachep;         slab cache, 用于__getname()
struct kmem_cache \*dentry_cache;  
struct kmem_cache \*inode_cache;
struct kmem_cache \*filp_cache;
struct kmem_cache \*mnt_cache;
========================================= ====================================

.. code-block:: c

    # 初始启动阶段
    start_kernel()              [main.c]
      vfs_caches_init_early()
        dcache_init_early()                       # early申请哈希表dir-cache, 
        inode_init_early()                        # early申请哈希表inode-cache, 
      vfs_caches_init(totalarm_pages)             # vfs cache初始化
        kmem_cache_create("names_cache")          # 创建names_cache
        dcache_init();
          mem_cache_create("dentry")              # 创建dentry cache
          alloc_large_system_hash("Dentry cache")
        inode_init();
          kmem_cache_create("inode_cache")        # 创建inode cache
          alloc_large_system_hash("inode-cache")  # 申请hash表
        files_init(totalarm_pages)
          kmem_cache_create("filp")
        mnt_init()
          kmem_cache_create("mnt_cache")   
          alloc_large_system_hash("Mount-cache")
          alloc_large_system_hash("Mountpoint-cache")
          kernfs_init()
            kmem_cache_create("kernfs_node_cache")
          sysfs_init()
            kernfs_create_root()
            register_filesystem(&sysfs_fs_type);
            kobject_create_and_add("fs", NULL);     # 添加fs文件夹在/sys目录下
          init_rootfs()
            register_filesystem(&rootfs_fs_type);   # 注册rootfs文件系统类型
          init_mount_tree()       
            get_fs_type("rootfs")
            
        bdev_cache_init()
        chrdev_init()
    
