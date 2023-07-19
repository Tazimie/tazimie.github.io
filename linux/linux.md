# linux

## 设置终端代理

```
alias proxy='export all_proxy=socks5://127.0.0.1:1080'
alias unproxy='unset all_proxy'
```

10.110.24.172 169.254.184.59 

## 查看你的ip

 curl cip.cc

## 看看容量

`sudo du --max-depth=1 -h`

` ls -hl` 如果文件不是很多也不是很大：

`df -hl` 查看磁盘容量

#### 重新挂载目录

1. 创建临时目录 `sudo mkdir /storage`
2. 把`/dev/sdb1`挂载到`/storage`，命令：`sudo mount /dev/sdb1 /storage`
3. 拷贝数据到临时目录：`cp -pdr /var/* /storage/`
4. 备份原目录：`sudo mv /var /var_old`
5. 新建/var：`sudo mkdir /var`
6. 取消临时目录挂载：`sudo umount /dev/sdb1`
7. 重新挂载`/dev/sdb1`到`/var`：`mount /dev/sdb1 /var`
8. 清理对于目录：`rm -rf /var/lost+found`
9. 设置开机自动挂载磁盘:

```shell
vim /etc/fstab
## 增加卷自动挂载配置
UUID=9fb0bd07-50df-46e1-8944-db25f3ac100a /var   ext4 defaults 0 2 
UUID=c684f54c-057d-4e4f-91c0-c028837c2c00 /home  ext4 defaults 0 2

## 其中uuid可以通过以下命令查看
blkid /dev/sdb1
```



![image-20220324154523271](https://gitee.com/tazimie/img/raw/master/uPic/image-20220324154523271.png)

https://www.jianshu.com/p/a2d773965de5

![img](https://gitee.com/tazimie/img/raw/master/uPic/webp.jpg)

## 修改权限命令

- chmod
- chown
- chgrp

## 软连接与硬链接的区别

“区别：不能对目录创建硬链接；可以对目录创建软链接，遍历操作会忽略目录的软链接。不能对不同的文件系统创建硬链接；可以跨文件系统创建软链接。不能对不存在的文件创建硬链接；可以对不存在的文件创建软链接。”

## 定时任务

`corntab`

## tmux

**鼠标模式**

set -g mouse on 

**会话(session)管理**

- 新建会话：tmux new -s
- 分离会话：tmux detach
- 查看会话：tmux ls
- 接入会话：tmux attach -t
- 切换会话：tmux switch -t
- 杀掉会话：tmux kill-session -t
- 重命名会话：tmux rename-session -t

**窗口(window)管理**

- 新建窗口：tmux new-window OR tmux new-window -n
- 切换窗口：tmux select-window -t
- 重命名窗口：tmux rename-window

**窗格(pane)管理**

- 划分窗格：tmux split-window

- 移动光标：tmux select-pane
- 交换窗格：tmux swap-pane

## ranger

配置`~/.config/ranger/rc.conf`

`S` for shell

`ctrl+d` for return

## fzf

`ctrl+r` search history

## &

首先，linux进程是区分前台进程和后台进程的。 
通常，在终端输入的命令执行的前台进程模式。如果一个命令要执行好久，就会阻塞住终端好久，不能进行其他工作，所以，我们可以把执行花费时间很长的任务使用后台进程模式运行，我们就可以在同一终端干其他事！、

### 以前台进程模式运行

通常使用的方式

```ruby
[root@localhost cdnjs]# find / -name xml &
```

### 以后台进程模式运行

```ruby
[root@localhost cdnjs]# find / -name xml &
```

这样，这个查找程序就会在后台运行。它运行的同时不影响你干别的事情。 
在后台运行时，找到符合的文件，还是会在终端中输出。

### 查看后台任务

```ruby
[root@localhost cdnjs]# jobs



[1]+  已停止               find / -name xml
```

### 切换前台/后台模式

#### 前台切后台

在运行命令后，有的时候忘记了在命令之后加上‘&’符号，又不愿意停下此命令重新改写。这是可以按[ctrl+z]，把当前程序切入后台。 
但是要注意此时在后台的这个程序是处于 Stopped 状态 
要继续执行的话，先使用jobs命令找出当前任务的jobId，然后按如下操作

```ruby
[root@localhost cdnjs]#bg 1
```

#### 后台切回前台

先使用jobs命令找出当前任务的jobId，然后按如下操作

```ruby
[root@localhost cdnjs]#fg 1
```

## nohup

不管是前台进程还是后台进程，在终端关闭的时候，linux会发出终端关闭信号，让在终端中运行的进程结束。 
但是，我们可能会有这样的需求： 
在linux进行下载很久的任务，但是终端关闭的时候，我们是不希望下载被终止的。所以，可以采用nohup命令的方式，让程序运行的时候，忽略掉终端关闭的信号。 
格式为：

```bash
nohup 执行程序的命令
```

### tip

如果想让一个程序在后台运行，只要在执行命令的末尾加上一个&符号就可以了。但是这种方式不是很保险，有些程序当你登出终端后它就会停止。那么如何让一个程序真正永远在后台执行呢。答案就是使用 nohup和&组合使用 
格式为：

```bash
nohup 执行程序的命令 &
```

### screen

nohup和&的缺点是，如果你要在一个shell会话里面执行多个命令和脚本，那么要每个命令和脚本都要加nohup和&非常麻烦，所以才有了screen。



## 使用bash 的[作业控制](http://web.mit.edu/gnu/doc/html/features_5.html)将流程发送到后台：

1. Ctrl+ Z停止（暂停）程序并返回外壳。
2. `bg` 在后台运行它。
3. `disown -h [job-spec]`其中，[job-spec]是作业编号（如`%1`第一个正在运行的作业；请使用`jobs`命令查找您的编号），以便在终端关闭时不会终止该作业。
