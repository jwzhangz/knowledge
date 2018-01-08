基于磁盘的文件系统，使用面向块的介质。需要解决如下问题，如何将文件内容和结构信息存储在目录层次结构上。从文件系统角度看，底层设备无非是存储块组成的一个列表，文件系统相当于对该列表实施一个适当的组织方案。


文件描述符是在打开文件时由内核分配，只在一个进程内有效。

内核处理文件的关键是inode。每个文件和目录都有且只有一个inode，其中包含元数据(访问权限等)和指向文件数据的指针。

inode的成员分为两类：
描述文件状态的元数据。
保存实际文件内容的数据段。

内核查找 /usr/bin/emacs 的inode的过程。

查找起始 inode，表示根目录/，对系统来说必须是已知的。其数据段不包含普通数据，而是根目录下的各个目录项。这些目录项代表文件或其他目录。每一项由两个成员组成。inode编号和文件或目录名称。

系统中inode编号唯一。查找usr的inode。通过根目录的inode接着获取对应的数据段。找到目录名为usr的inode。重复这个过程。

#### 链接
两种类型：符号链接和硬链接。

###### 符号链接
又称软链接。是一个目录项，包含目标文件的文件名。目标文件删除时，软链接并不会被删除。符号链接也有一个inode。

###### 硬链接
硬链接建立后，无法区分哪个文件是原来的，哪个是后来建立的。inode中包含一个计数器，创建硬链接时，计数器加1.如果硬链接或原始文件被删除，计数器减1。如果计数器为0，则完全删除文件。

#### 编程接口
用户进程使用open和openat系统调用打开文件。内核向用户层返回一个非负整数。从3开始。0表示标准输入，1为标准输出，2为标准错误输出。

传统上文件描述符在内核中足以标识一个文件。由于多命名空间和容器的引入，多个文件的描述符可以具有相同值。数据结构struct file唯一表示一个文件。

 
谁来创建super block，什么时候创建？super block对应着一个挂载点。
 
文件系统装载由mount触发。

#### 阅读代码
[Linux中__init、__devinit等初始化宏](http://blog.csdn.net/yinwei520/article/details/6646933)
 
[Linux的 __setup解析 -- 命令行处理](http://blog.csdn.net/wh_19910525/article/details/42779943)
 
dentry 是什么？

```c
struct hlist_bl_node d_hash;
父目录的dentry
struct dentry *d_parent; 

该目录或文件?name
struct qstr d_name;
对应的inode
struct inode *d_inode;

dentry操作函数
const struct dentry_operations *d_op;

所在根节点的super block
struct super_block *d_sb;

parent的子目录
struct list_head d_child;
自己的所有子目录
struct list_head d_subdirs;
```
[Linux 的虚拟文件系统(强烈推荐)](http://blog.csdn.net/heikefangxian23/article/details/51579971)

[[文件系统]文件系统学习笔记（七）----pathwalk(2)](https://www.cnblogs.com/zhiliao112/p/4067844.html)


当用户输入”mount /dev/sdb /mnt/alan”命令后，Linux会解析/mnt/alan字符串，并且从Dentry Hash表中获取相关的dentry目录项，然后将该目录项标识成DCACHE_MOUNTED。一旦该dentry被标识成DCACHE_MOUNTED，也就意味着在访问路径上对其进行了屏蔽。
 
在mount /dev/sdb设备上的ext3文件系统时，内核会创建一个该文件系统的superblock对象，并且从/dev/sdb设备上读取所有的superblock信息，初始化该内存对象。Linux内核维护了一个全局superblock对象链表。s_root是superblock对象所维护的dentry目录项，该目录项是该文件系统的根目录。即新mount的文件系统内容都需要通过该根目录进行访问。在mount的过程中，VFS会创建一个非常重要的vfsmount对象，该对象维护了文件系统mount的所有信息。Vfsmount对象通过HASH表进行维护，通过path地址计算HASH值，在这里vfsmount的HASH值通过“/mnt/alan”路径字符串进行计算得到。Vfsmount中的mnt_root指向superblock对象的s_root根目录项。因此，通过/mnt/alan地址可以检索VFSMOUNT Hash Table得到被mount的vfsmount对象，进而得到mnt_root根目录项。
 
例如，/dev/sdb被mount之后，用户想要访问该设备上的一个文件ab.c，假设该文件的地址为：/mnt/alan/ab.c。在打开该文件的时候，首先需要进行path解析。在解析到/mnt/alan的时候，得到/mnt/alan的dentry目录项，并且发现该目录项已经被标识为DCACHE_MOUNTED。之后，会采用/mnt/alan计算HASH值去检索VFSMOUNT Hash Table，得到对应的vfsmount对象，然后采用vfsmount指向的mnt_root目录项替代/mnt/alan原来的dentry，从而实现了dentry和inode的重定向。在新的dentry的基础上，解析程序继续执行，最终得到表示ab.c文件的inode对象。


The dentry sequence count protects us from concurrent renames, and thus protects parent and name fields.
   
[dentry与inode、dentry_cache](http://blog.chinaunix.net/uid-30226-id-2441814.html)

文件有dentry吗？

[Linux内存管理原理](https://www.cnblogs.com/zhaoyl/p/3695517.html)

init_mount_tree

文件存储在硬盘中的状态。
