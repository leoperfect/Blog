## NFS-mount 跨服务器文件挂载

### 一、什么是NFS？

> `NFS`:`network file system`，网络文件系统，允许服务器之间通过TCP/IP协议进行资源共享。NFS客户端可以透明的读写NFS服务器上的文件，就像操作本地文件一样。

### 二、为什么要用NFS？NFS什么好处？

1. 节省空间：客户端磁盘空间较少，可以挂载到另外的服务器上，以节省本地存储空间。
2. 网络受限：有些公司内部服务器无法访问外网，但是一些操作需要用到外网权限，就可以将公司服务器挂载到可以访问外网的服务器上，在另外的服务器上进行操作。

### 三、怎么挂载呢？

场景：服务器A的`/mnt`目录 挂载到 服务器B上的`/test`目录上

#### 配置服务器A

1. 需要检查是否具有nfs服务

```shell
$ ls -al /etc/init.d/nfs-kernel-server // 查看是否存在nfs服务
```

如果没有 需要手动安装 nfs-kernel-server 服务

```shell
$ sudo apt-get install nfs-kernel-server
```

2. 修改 `/etc/exports`文件（需要root权限），增加要挂载的目录 `/mnt *(rw,sync)`

> 其中`/mnt`是要被挂在的目录，`*`表示任何服务器，也可以写客户端的IP地址，`(rw,sync)`表示挂载文件系统时的策略，`rw`表示读写，`sync`表示同步进行IO操作，还有其他的一些选项`async`(非同步进行IO操作)。

3、重启nfs服务

```shell
$ sudo /etc/init.d/nfs-kernel-server restart 
```

#### 配置服务器B

以root权限执行下面命令进行挂载

```
$ sudo mount -t nfs 10.24.21.143:/mnt /test
```

> `-t nfs` 表示挂载类型是nfs，`10.24.21.143:/mnt`表示服务器A的IP及需要被挂载的目录，`/test`表示挂载到服务器B的目录。

执行下列命令查看是否已经挂载成功

```shell
$ mount | grep nfs // 如果成功，能够看到挂载的信息
```

### 四、配置过程中遇到的坑

其中在服务器B进行挂载时遇到报错如下：

```she
mount: wrong fs type, bad option, bad superblock on 10.24.21.143:/mnt,
       missing codepage or helper program, or other error
       (for several filesystems (e.g. nfs, cifs) you might
       need a /sbin/mount.<type> helper program)

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
```

错误信息中提到

```
you might need a /sbin/mount.<type> helper program
```

指在mount过程中用到了 `/sbin/mount.nfs`程序，而`/sbin/mount.nfs`是`nfs-common`提供的，需要手动运行下面的命令安装一下就好了。

```shell
$ sudo apt-get install nfs-common
```

