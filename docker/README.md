# Container

Container主要有Namespace，Cgroup, Layered FileSystem组成

## Namespace
### UTS
UTS namespace 功能最简单，它只隔离了 hostname 和 NIS domain name 两个资源。同一个 namespace 里面的进程看到的 hostname 和 domain name 是相同的，这两个值可以通过 sethostname(2) 和 setdomainname(2) 来进行设置，也可以通过 uname(2)、gethostname(2) 和 getdomainname(2) 来读取
### PID
PID namespace 隔离的是进程的 pid 属性，也就是说不同的 namespace 中的进程可以有相同的 pid。PID namespace 和我们常见的系统规则一样，都是从 pid 1 开始，每次 fork、vfork、clone 调用都会分配新的 pid。  
在传统的UNIX系统中，PID为1的进程是init，地位非常特殊。他作为所有进程的父进程，有很多特权（比如：屏蔽信号等），另外，其还会为检查所有进程的状态，我们知道，如果某个子进程脱离了父进程（父进程没有wait它），那么init就会负责回收资源并结束这个子进程。所以，要做到进程空间的隔离，首先要创建出PID为1的进程，最好就像chroot那样，把子进程的PID在容器内变成1。  
但是，我们会发现，在子进程的shell里输入ps,top等命令，我们还是可以看得到所有进程。说明并没有完全隔离。这是因为，像ps, top这些命令会去读/proc文件系统，所以，因为/proc文件系统在父进程和子进程都是一样的，所以这些命令显示的东西都是一样的。  
所以，我们还需要对文件系统进行隔离
### MNT
Mount namespace 隔离的是 mount points（挂载点），也就是说不同 namespace 下面的进程看到的文件系统结构是不同的，namespace 内的 mount points 可以通过 mount(2) 和 umount(2) 来修改，因此Mount namespace 可以用来实现容器文件系统的隔离
```
syscall.Mount("proc", "proc", "proc", 0, "")
```

### NET
Net namespace 隔离的是和网络相关的资源，包括网络设备、路由表、防火墙(iptables)、socket（ss、netstat）、 /proc/net 目录、/sys/class/net 目录、网络端口(network interfaces)等等。

一个物理网络设备只能出现在最多一个网络 namespace 中，不同网络 namespace 之间可以通过创建 veth pair 提供类似管道的通信
### USER
User namespace 隔离的是用户和组信息，在不同的 namespace 中用户可以有相同的 UID 和 GID，它们之间互相不影响。另外，还有父子 namespace 之间用户和组映射的功能。父 namespace 中非 root 用户也能成为子 namespace 中的 root，这样就能增加安全性（如果所有 namespace 的 root 用户都是一样的，会带来子 namespace 操作父 namespace 内容的危险）。

要想实现容器内部和外部用户/组的 mapping，要把对应关系写入到 /proc/PID/uid_map 和 /proc/PID/gid_map 文件中，PID 表示容器内进程在父 namespace 中的 PID（如果没有 PID 隔离，也就是容器内部的 PID）
### IPC
IPC（Inter-Process Communication）命名空间的主要作用是隔离不同进程之间的进程间通信资源（信号量，消息队列，共享内存段）。通过将这些资源划分到不同的命名空间中，可以防止不同应用或服务之间的干扰，提高系统的安全性和稳定性。  
创建消息队列
```
ipcmk -Q
```
查看所有 IPC 资源:
```
vagrant@ubuntu-bionic:~$ ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x95da5267 0          vagrant    644        0            0           

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      

------ Semaphore Arrays --------
key        semid      owner      perms      nsems 

```
只查看消息队列:
```
vagrant@ubuntu-bionic:~$ ipcs -q 

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x95da5267 0          vagrant    644        0            0           

```

# Reference
+ https://www.infoq.com/articles/build-a-container-golang/  
+ https://www.youtube.com/watch?v=8fi7uSYlOdc
+ https://cizixs.com/2017/08/29/linux-namespace/
+ https://coolshell.cn/articles/17010.html