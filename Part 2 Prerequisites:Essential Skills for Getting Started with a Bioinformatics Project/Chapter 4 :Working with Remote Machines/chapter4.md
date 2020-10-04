第四章 登陆远程服务器工作
生物信息学中的大多数数据处理任务和我们的工作站的相比需要更多的计算能力
，这意味着我们必须使用大型服务器或计算机集群。对于某些生物信息学项目，您可能会主要从事
通过与远程计算机的网络连接。毫不奇怪，与远程机器对于初学者来说可能非常令人沮丧，并且可
能会降低有经验的生物信息学家效率。在本章中，我们将学习如何制作尽可能轻松地使用远程计算机，
以便您可以集中精力和项目本身的努力。

使用SSH远程连接
通过网络有很多方法连接远程服务器。但是目前最常用的是 secure shell （SSH）.我们使用SSH
是因为它已加密（这样可以安全地发送密码，编辑私人文件等),还有一个原因就是它在每个机器上都有。
如何配置服务器，SSH和用户帐户您或系统管理员确定的内容；本章将不介绍这些系统管理主题。
本节介绍的材料应该会有所帮助您可以回答系统管理员可能会问的常见SSH问题（例如，“您有SSH吗公钥？”).
 你还将学习作为生物信息学家所需要的所有基本知识SSH连接到远程计算机。
 要初始化到主机的SSH连接（在本例中，biocluster.myuniversity.edu)，我们使用ssh命令：
 ```bash 
$ ssh biocluster.myuniversity.edu
Password:
Last login: Sun Aug 11 11:57:59 2013 from fisher.myisp.com
wsobchak@biocluster.myuniversity.edu$
 ```
 第一行是使用SSH连接远程主机，你会被提示输入远程的账户和密码
 第二行在你使用密码登陆之后，你会看到shell提示符，证明你可以想本地机器一样输入命令进行操作。
 SSH也支持IP地址的。例如，你想连接一个IP是192.169.237.42的机器，如果你的端口不是默认的22，
 或者你的远程的用户名和当前的用户名不一样，你必须使用下面的命令进行指定：
 ```bash 
$ ssh -p 50453 cdarwin@biocluster.myuniversity.edu
 ```
 这里我们通过-p指定端口，user@domain指定用户名。如果你不能链接到一个host,可以使用-v (-v for verbose)
 能帮你发现问题。可以通过使用-vv或-vvv来增加详细程度；请参阅man ssh获取更多详细信息。

    保存你常用的SHH host.
    生物信息学家经常需要通过SSH连接到服务器，然后输入输出IP地址或长域名可能变得相当乏味。
另外，记住并键入额外的内容也是很麻烦的详细信息，如远程服务器的端口或您的远程用户名。
SSH背后的开发人员创建了一个聪明的替代方案：SSH配置文件。SSH配置文件存储您经常连接的主机
的详细信息到。SSH配置文件存储有关您经常连接到的主机的详细信息。这个文件很容易创建，存储在
这个文件中的主机可以工作不仅是ssh，还有我们将在第6章中学习的两个程序：scp和rsync。
要创建一个文件，只需在~/.ssh/config处创建并编辑该文件。每个条目采用以下形式：
```bash 
Host bio_serv
HostName 192.168.237.42
User cdarwin
Port 50453
```
您不需要指定端口和用户，除非它们与远程主机的默认值。保存此文件后，您可以通过进入192.168.236.42
使用别名ssh bio_serv而不是键入ssh -p 50453cdarwin@192.169.237.42。

如果您在许多终端选项卡中使用许多远程计算机连接，那么确保您在您认为自己所在的主机上工作是很有用的。
你可以使用hostname命令访问主机名：
```bash 
$ hostname
biocluster.myuniversity.edu
```
同样，如果在服务器上有多个账户，你也可以确定你是哪一个用户在登陆。命令 whoami 返回使用者的名称。
```bash 
$ whoami
cdarwin
```
如果您偶尔使用管理员帐户登录，这一点尤其有用如果有更多的特权，在具有管理员权限的帐户要高得多，
因此您应该始终离开当你在这个帐户上时（尽可能减少这个时间）。

快速身份认证
SSH要求您为远程计算机上的帐户键入密码。但是，每次登录时输入密码可能会很乏味，而且并不总是安全的。
一个更安全、更简单的选择是使用SSH公钥。公钥密码术是一项迷人的技术，但细节是超出了本书的范围。

    使用SSH密钥登录到远程计算机如果没有密码，我们首先需要生成一个公钥/私钥对。我们使用ssh-keygen
命令执行此操作。注意您的公钥和私钥之间的区别是非常重要的：您可以将您的公钥分发给其他人服务器，但你的
私钥必须保持安全，绝不共享。
    让我们使用SSH-keygen生成一个SSH密钥对：
```bash 
$ ssh-keygen -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/username/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/username/.ssh/id_rsa.
Your public key has been saved in /Users/username/.ssh/id_rsa.pub.
The key fingerprint is:
e1:1e:3d:01:e1:a3:ed:2b:6b:fe:c1:8e:73:7f:1f:f0
The key's randomart image is:
+--[ RSA 2048]----+
|.o... ... |
| . . o |
| . * |
| . o + |
| . S . |
| o . E |
| + . |
|oo+.. . . |
|+=oo... o. |
+-----------------+
```
这将在~/.ssh/id_rsa创建一个私钥，在~/.ssh/id处创建一个公钥_rsa.pub.
sshkeygen为您提供了使用空密码的选项，但通常建议您使用真实密码。如果您想知道，
sshkeygen创建的随机是验证您的密钥的一种方法（如果您好奇的话，在man ssh中有更多的细节）。


使用nohup和tmux获取长时间运行的任务
在第三章中，我们简要介绍了进程（不管是在前端还是后端运行的）会在我们呢关闭命令行窗口后中断。
进程也会在我们与远程服务器断开之后中断，这个操作时故意的，因为你的程序会挂断，导致你的程序
不正常退出。因为生信每天都登陆远程服务器工作，所以必须找到一种方法阻止这种情况发生。有两种
解决方案：nohup 和 Tmux.
nohup是执行命令并捕捉发送的挂断信号的简单命令从终点站。因为nohup命令捕捉并忽略这些挂断信号，
你正在运行的程序不会被中断。运行命令使用nohup就像在命令前添加nohup一样简单：
```bash 
$ nohup program1 > output.txt &
[1] 10900
```



