---
layout: post
title: Linux内核分析(4) 文件系统--inode剖析
categories: [linux kernel ]
tags: [linux kernel, ]
description: linux 的文件系统 struct inode
---


## what is inode?

文件存储在硬盘上，硬盘的最小存储单位叫做“扇区”（sector），每个扇区512字节。

操作系统读取硬盘的时候，不会一个个扇区的读取，这样效率太低，而是一次性读取多个扇区，即一个“块”（block）。这种由多个扇区组成的“块”，是文件存取的最小单位。“块”的大小，最常见的是4KB。

文件数据储存在“块”中，此外我们还需要一个地方存储文件的元信息，例如文件的创建者、创建日期、修改日期、文件大小等。这种存储文件元信息的区域就叫做inode，中文译名“索引节点”。

每一个文件都有对应的inode，里面包含了与该文件有关的一些信息。

大部分的文件系统，如Ext、Fat、Ntfs等，都会在磁盘格式化的时候创建inode区，存放文件的inode信息。

Linux的VFS子系统，会对磁盘中的inode信息做一个封装，形成VFS中定义的inode结构体。

## inode的内容

磁盘中的inode包含文件的元信息，具体说来有以下内容：

```
* 文件的字节数
* 文件拥有者的User ID
* 文件的Group ID
* 文件的读、写、执行权限
* 文件的时间戳，ctime、mtime、atime
* 链接数
* 文件的大小
* 文件数据的block位置
* 文件数据的block个数
```

下面来看下linux kernel中的inode定义，位于 linux/include/linux/fs.h：

```c
struct address_space {
	struct inode		*host;		/* owner: inode, block_device */
	struct radix_tree_root	page_tree;	/* radix tree of all pages */
	spinlock_t		tree_lock;	/* and lock protecting it */
	atomic_t		i_mmap_writable;/* count VM_SHARED mappings */
	struct rb_root		i_mmap;		/* tree of private and shared mappings */
	struct rw_semaphore	i_mmap_rwsem;	/* protect tree, count, list */
	/* Protected by tree_lock together with the radix tree */
	unsigned long		nrpages;	/* number of total pages */
	/* number of shadow or DAX exceptional entries */
	unsigned long		nrexceptional;
	pgoff_t			writeback_index;/* writeback starts here */
	const struct address_space_operations *a_ops;	/* methods */
	unsigned long		flags;		/* error bits/gfp mask */
	spinlock_t		private_lock;	/* for use by the address_space */
	struct list_head	private_list;	/* ditto */
	void			*private_data;	/* ditto */
} __attribute__((aligned(sizeof(long))));
	/*
	 * On most architectures that alignment is already the case; but
	 * must be enforced here for CRIS, to let the least significant bit
	 * of struct page's "mapping" pointer be used for PAGE_MAPPING_ANON.
	 */

/*
 * Keep mostly read-only and often accessed (especially for
 * the RCU path lookup and 'stat' data) fields at the beginning
 * of the 'struct inode'
 */
struct inode {
	umode_t			i_mode;
	unsigned short		i_opflags;
	kuid_t			i_uid;
	kgid_t			i_gid;
	unsigned int		i_flags;

	const struct inode_operations	*i_op;
	struct super_block	*i_sb;
	struct address_space	*i_mapping;

	/* Stat data, not accessed from path walking */
	unsigned long		i_ino;
	/*
	 * Filesystems may only read i_nlink directly.  They shall use the
	 * following functions for modification:
	 *
	 *    (set|clear|inc|drop)_nlink
	 *    inode_(inc|dec)_link_count
	 */
	union {
		const unsigned int i_nlink;
		unsigned int __i_nlink;
	};
	dev_t			i_rdev;
	loff_t			i_size;
	struct timespec		i_atime;
	struct timespec		i_mtime;
	struct timespec		i_ctime;
	spinlock_t		i_lock;	/* i_blocks, i_bytes, maybe i_size */
	unsigned short          i_bytes;
	unsigned int		i_blkbits;
	blkcnt_t		i_blocks;

	/* Misc */
	unsigned long		i_state;
	struct rw_semaphore	i_rwsem;

	unsigned long		dirtied_when;	/* jiffies of first dirtying */
	unsigned long		dirtied_time_when;

	struct hlist_node	i_hash;
	struct list_head	i_io_list;	/* backing dev IO list */

	struct list_head	i_lru;		/* inode LRU list */
	struct list_head	i_sb_list;
	union {
		struct hlist_head	i_dentry;
		struct rcu_head		i_rcu;
	};
	u64			i_version;
	atomic_t		i_count;
	atomic_t		i_dio_count;
	atomic_t		i_writecount;

	const struct file_operations	*i_fop;	/* former ->i_op->default_file_ops */
	struct file_lock_context	*i_flctx;
	struct address_space	i_data;
	struct list_head	i_devices;
	union {
		struct pipe_inode_info	*i_pipe;
		struct block_device	*i_bdev;
		struct cdev		*i_cdev;
		char			*i_link;
		unsigned		i_dir_seq;
	};

	__u32			i_generation;

	void			*i_private; /* fs or device private pointer */
};
```

[Linux的inode的理解](http://www.cnblogs.com/itech/archive/2012/05/15/2502284.html)
[理解inode](http://www.ruanyifeng.com/blog/2011/12/inode.html)
[Linux内核源代码情景分析-第五章 文件系统](http://blog.sina.com.cn/s/blog_6b94d5680101vfqv.html)
[史上最经典的Linux内核学习方法论](http://blog.chinaunix.net/uid-26258259-id-3783679.html)
