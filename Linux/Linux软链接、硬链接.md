## 1. 关系图

<img src="https://xzchsia.github.io/img/in-post/linux-hard-soft-link/linux-soft-hard-link-diff.png" alt="Linux硬链接和软连接的区别" style="zoom: 67%;" />

1. linux使用ext4文件系统，他把磁盘分成两块：小部分为inode、大部分为block
2. inode存储文件的属性（包括文件的权限（r、w、x）、文件的所有者和属组、文件的大小、文件的状态改变时间（ctime）、文件的最近一次读取时间（atime）、文件的最近一次修改时间（mtime）、文件的数据真正保存的 block 编号）
3. 文件的数据真正保存在block中
4. 硬链接与源文件指向同一个inode
5. 软链接指向源文件，可理解成源文件的快捷方式，而源文件指向inode

## 2. 创建

1. 硬链接

```bash
ln file1 file2	#创建file1的硬链接file2
```

2. 软链接

```bash
ln -s file1 file2	#创建file1的软链接file2
```