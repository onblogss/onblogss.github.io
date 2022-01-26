---
layout: post
title: 使用SSH+SFTP操作终端全解析，告别XShell
date: 2019-07-10 20:46:00
author: 薛师兄
tags:
  - SSH
  - SFTP
---
## 1.前言

在Windows系统下操作远程服务器的方式很多，比如XShell+XFTP组合，亦或是PuTTY+WinSCP组合，但在Mac系统下登陆远程服务器，并没有这些工具供我们使用。相比较而言，在Mac下更多的是依赖终端输入SSH命令登陆远程服务器。

使用SSH命令行的好处就是可以近距离接触底层，用的越多，用的越溜，对SSH的原理就越了解。相反，使用现成的SSH工具（PuTTY、XShell），我们其实并不会有涉及使用ssh命令的机会，对大多数人而言，怕是只知道最基本的`ssh root@ip`。

本文将带大家了解ssh的原理与使用技巧，帮助更多终端爱好者更方便更随心所欲的使用终端。

## 2.SSH是什么

SSH服务其实是一个守护进程(demon)，系统后台会监听客户端的连接，ssh服务端的进程名为sshd，负责实时监听客户端的请求(IP 22端口)，包括公共秘钥等交换等信息。SSH服务端由2部分组成： openssh(提供ssh服务)、openssl(提供加密的程序)。

## 3.对称加密和非对称加密

在学习SSH的工作机制之前，我们需要了解对称加密和非对称加密的原理。

**对称加密**

所谓对称加密，是采用对称密码编码技术的加密措施，它的特点是文件加密和解密都是使用相同的密钥。

这种方法在密码学中叫做对称加密算法，对称加密算法使用起来简单快捷，密钥较短，且破译困难，除了数据加密标准（DES），另一个对称密钥加密系统是国际数据加密算法（IDEA），它比DES的加密性好，而且对计算机功能要求也没有那么高。

**非对称加密**

与对称加密算法不同，非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。

公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。

因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

## 3.SSH如何工作

了解了对称加密和非对称加密是什么之后，再来了解SSH如何使用非对称加密技术，大致流程如下：

在服务器启动的时候会产生一个密钥(也就是768bit公钥)，本地的ssh客户端发送连接请求到ssh服务器，服务器检查连接点客户端发送的数据和IP地址，确认合法后发送密钥(768bits公钥)给客户端，此时客户端将本地私钥(256bit)和服务器的公钥(768bit)结合成密钥对key(1024bit)，发回给服务器端，服务端利用自己的私钥解密，读取出客户端的本地私钥，建立连接通过key-pair数据传输，在此之后，服务端与客户端就愉快的使用客户端私钥进行沟通。

## 3.SSH命令详解

SSH命令最简单的用法只需要指定用户名和主机名参数即可，主机名可以是 IP 地址或者域名。

```shell
ssh root@192.168.0.1
```

**指定端口号**

SSH 默认连接到目标主机的 22 端口上，我们可以使用 -p 选项指定端口号。

```shell
ssh -p 22 root@192.168.0.1
```

**追加命令**

使用 SSH 在远程主机执行一条命令并显示到本地，然后继续本地工作，只需要直接连接并在后面加上要执行的命令。

```shell
ssh -p 22 root@192.168.0.1 ls -l
```

**图形界面**

在远程主机运行一个图形界面的程序，只需使用SSH的-X选项，然后主机就会开启 X11转发功能。

```shell
ssh -X 22 root@192.168.0.1
```

**绑定源地址**

如果你的客户端有多于两个以上的 IP 地址，你就不可能分得清楚在使用哪一个 IP 连接到 SSH 服务器。为了解决这种情况，我们可以使用 -b 选项来指定一个IP 地址。这个 IP 将会被使用做建立连接的源地址。

```shell
ssh -b 192.168.0.200 root@192.168.0.103
```

**对所有数据请求压缩**

使用 -C 选项，所有通过 SSH 发送或接收的数据将会被压缩，并且仍然是加密的。 

```shell
ssh -C root@192.168.0.103
```

**打开调试模式**

因为某些原因，我们想要追踪调试我们建立的 SSH 连接情况。SSH 提供的 -v 选项参数正是为此而设的。其可以看到在哪个环节出了问题。

```shell
ssh -v root@192.168.0.103
```

## 4.SSH免密登陆

通过SSH命令登陆远程服务器需要手动的每次输入密码，解决这个问题其实非常简单，通过 ssh-keygen 生成本地公钥和私钥，将公钥Copy到远程服务器就可以。

**1.构建 SSH 密钥对**

使用 ssh-keygen -t +算法名，现在大多数都使用 RSA 或者 DSA 算法。

如果你在安装Git时已经做过此步骤，那么忽略这一步即可。

```shell
ssh-keygen -t rsa
```

**2.拷贝本地公钥给远程服务器**

```shell
ssh-copy-id root@192.168.25.110 
```

你可以通过参数 -i 指定公钥文件

```shell
ssh-copy-id -i id_dsa.pub omd@192.168.25.110
```

**3.查看是否已经添加了对应主机的密钥**

使用 -F 选项

```shell
ssh-keygen -F 192.168.0.1
```

**4.删除主机密钥**

使用-R选项，也可以在 ~/.ssh/known_hosts 文件中手动删除

```shell
ssh-keygen -R 192.168.0.1
```

## 5.如何配置 SSH

SSH 的配置文件在 /etc/ssh/sshd_config 中，你可以看到端口号，空闲超时时间等配置项。

```shell
cat /etc/ssh/sshd_config
```

/etc/ssh/sshd_config 配置文件详细说明

```yaml
#############1. 关于 SSH Server 的整体设定##############
#Port 22    
##port用来设置sshd监听的端口，为了安全起见，建议更改默认的22端口为5位以上陌生端口
#Protocol 2,1
Protocol 2
##设置协议版本为SSH1或SSH2，SSH1存在漏洞与缺陷，选择SSH2
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress用来设置sshd服务器绑定的IP地址
##监听的主机适配卡，举个例子来说，如果您有两个 IP， 分别是 192.168.0.11 及 192.168.2.20 ，那么只想要
###开放 192.168.0.11 时，就可以设置为：ListenAddress 192.168.0.11
####表示只监听来自 192.168.0.11 这个 IP 的SSH联机。如果不使用设定的话，则预设所有接口均接受 SSH

#############2. 说明主机的 Private Key 放置的档案##########　　　　　　　　　　　　　　　　　
#ListenAddress ::
##HostKey用来设置服务器秘钥文件的路径
# HostKey for protocol version 1
#HostKey /etc/ssh/ssh_host_key
##设置SSH version 1 使用的私钥

# HostKeys for protocol version 2
#HostKey /etc/ssh/ssh_host_rsa_key
##设置SSH version 2 使用的 RSA 私钥

#HostKey /etc/ssh/ssh_host_dsa_key
##设置SSH version 2 使用的 DSA 私钥


#Compression yes　　　　　　
##设置是否可以使用压缩指令

# Lifetime and size of ephemeral version 1 server key
#KeyRegenerationInterval 1h
##KeyRegenerationInterval用来设置多长时间后系统自动重新生成服务器的秘钥，
###（如果使用密钥）。重新生成秘钥是为了防止利用盗用的密钥解密被截获的信息。

#ServerKeyBits 768
##ServerKeyBits用来定义服务器密钥的长度
###指定临时服务器密钥的长度。仅用于SSH-1。默认值是 768(位)。最小值是 512 。


# Logging
# obsoletes QuietMode and FascistLogging
#SyslogFacility AUTH
SyslogFacility AUTHPRIV
##SyslogFacility用来设定在记录来自sshd的消息的时候，是否给出“facility code”

#LogLevel INFO
##LogLevel用来设定sshd日志消息的级别


#################3.安全认证方面的设定################
#############3.1、有关安全登录的设定###############
# Authentication:
##限制用户必须在指定的时限内认证成功，0 表示无限制。默认值是 120 秒。

#LoginGraceTime 2m
##LoginGraceTime用来设定如果用户登录失败，在切断连接前服务器需要等待的时间，单位为妙

#PermitRootLogin yes
##PermitRootLogin用来设置能不能直接以超级用户ssh登录，root远程登录Linux很危险，建议注销或设置为no

#StrictModes yes
##StrictModes用来设置ssh在接收登录请求之前是否检查用户根目录和rhosts文件的权限和所有权，建议开启
###建议使用默认值"yes"来预防可能出现的低级错误。

#RSAAuthentication yes
##RSAAuthentication用来设置是否开启RSA密钥验证，只针对SSH1

#PubkeyAuthentication yes
##PubkeyAuthentication用来设置是否开启公钥验证，如果使用公钥验证的方式登录时，则设置为yes

#AuthorizedKeysFile     .ssh/authorized_keys
##AuthorizedKeysFile用来设置公钥验证文件的路径，与PubkeyAuthentication配合使用,默认值是".ssh/authorized_keys"。
###该指令中可以使用下列根据连接时的实际情况进行展开的符号： %% 表示'%'、%h 表示用户的主目录、%u 表示该用户的用户名
####经过扩展之后的值必须要么是绝对路径，要么是相对于用户主目录的相对路径。

 
#############3.2、安全验证的设定###############
# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#RhostsRSAAuthentication no
##是否使用强可信主机认证(通过检查远程主机名和关联的用户名进行认证)。仅用于SSH-1。
###这是通过在RSA认证成功后再检查 ~/.rhosts 或 /etc/hosts.equiv 进行认证的。出于安全考虑，建议使用默认值"no"。

# similar for protocol version 2
#HostbasedAuthentication no
##这个指令与 RhostsRSAAuthentication 类似，但是仅可以用于SSH-2。

# Change to yes if you don't trust ~/.ssh/known_hosts for
# RhostsRSAAuthentication and HostbasedAuthentication

#IgnoreUserKnownHosts no
##IgnoreUserKnownHosts用来设置ssh在进行RhostsRSAAuthentication安全验证时是否忽略用户的“/$HOME/.ssh/known_hosts”文件
# Don't read the user's ~/.rhosts and ~/.shosts files

#IgnoreRhosts yes
##IgnoreRhosts用来设置验证的时候是否使用“~/.rhosts”和“~/.shosts”文件

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
##PasswordAuthentication用来设置是否开启密码验证机制，如果用密码登录系统，则设置yes

#PermitEmptyPasswords no
#PermitEmptyPasswords用来设置是否允许用口令为空的账号登录系统，设置no

#PasswordAuthentication yes
##是否允许使用基于密码的认证。默认为"yes"。
PasswordAuthentication yes

# Change to no to disable s/key passwords
##设置禁用s/key密码
#ChallengeResponseAuthentication yes
##ChallengeResponseAuthentication 是否允许质疑-应答(challenge-response)认证
ChallengeResponseAuthentication no


########3.3、与 Kerberos 有关的参数设定，指定是否允许基于Kerberos的用户认证########
#Kerberos options
#KerberosAuthentication no
##是否要求用户为PasswdAuthentication提供的密码必须通过Kerberos KDC认证，要使用Kerberos认证，
###服务器必须提供一个可以校验KDC identity的Kerberos servtab。默认值为no

#KerberosOrLocalPasswd yes
##如果Kerberos密码认证失败，那么该密码还将要通过其他的的认证机制，如/etc/passwd
###在启用此项后，如果无法通过Kerberos验证，则密码的正确性将由本地的机制来决定，如/etc/passwd，默认为yes

#KerberosTicketCleanup yes
##设置是否在用户退出登录是自动销毁用户的ticket

#KerberosGetAFSToken no
##如果使用AFS并且该用户有一个Kerberos 5 TGT,那么开启该指令后，
###将会在访问用户的家目录前尝试获取一个AFS token,并尝试传送 AFS token 给 Server 端，默认为no

 

####3.4、与 GSSAPI 有关的参数设定，指定是否允许基于GSSAPI的用户认证，仅适用于SSH2####
##GSSAPI 是一套类似 Kerberos 5 的通用网络安全系统接口。
###如果你拥有一套 GSSAPI库，就可以通过 tcp 连接直接建立 cvs 连接，由 GSSAPI 进行安全鉴别。

# GSSAPI options
#GSSAPIAuthentication no
##GSSAPIAuthentication 指定是否允许基于GSSAPI的用户认证，默认为no

GSSAPIAuthentication yes
#GSSAPICleanupCredentials yes
##GSSAPICleanupCredentials 设置是否在用户退出登录是自动销毁用户的凭证缓存
GSSAPICleanupCredentials yes

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication mechanism.
# Depending on your PAM configuration, this may bypass the setting of
# PasswordAuthentication, PermitEmptyPasswords, and
# "PermitRootLogin without-password". If you just want the PAM account and
# session checks to run without PAM authentication, then enable this but set
# ChallengeResponseAuthentication=no
#UsePAM no
##设置是否通过PAM验证
UsePAM yes

# Accept locale-related environment variables
##AcceptEnv 指定客户端发送的哪些环境变量将会被传递到会话环境中。
###[注意]只有SSH-2协议支持环境变量的传递。指令的值是空格分隔的变量名列表(其中可以使用'*'和'?'作为通配符)。
####也可以使用多个 AcceptEnv 达到同样的目的。需要注意的是，有些环境变量可能会被用于绕过禁止用户使用的环境变量。
#####由于这个原因，该指令应当小心使用。默认是不传递任何环境变量。

AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL
AllowTcpForwarding yes

##AllowTcpForwarding设置是否允许允许tcp端口转发，保护其他的tcp连接

#GatewayPorts no
##GatewayPorts 设置是否允许远程客户端使用本地主机的端口转发功能，出于安全考虑，建议禁止

 
#############3.5、X-Window下使用的相关设定###############

#X11Forwarding no
##X11Forwarding 用来设置是否允许X11转发
X11Forwarding yes

#X11DisplayOffset 10
##指定X11 转发的第一个可用的显示区(display)数字。默认值是 10 。
###可以用于防止 sshd 占用了真实的 X11 服务器显示区，从而发生混淆。
X11DisplayOffset 10

#X11UseLocalhost yes

 

#################3.6、登入后的相关设定#################

#PrintMotd yes
##PrintMotd用来设置sshd是否在用户登录时显示“/etc/motd”中的信息，可以选在在“/etc/motd”中加入警告的信息

#PrintLastLog yes
#PrintLastLog 是否显示上次登录信息

#TCPKeepAlive yes
##TCPKeepAlive 是否持续连接，设置yes可以防止死连接
###一般而言，如果设定这项目的话，那么 SSH Server 会传送 KeepAlive 的讯息给 Client 端，以确保两者的联机正常！
####这种消息可以检测到死连接、连接不当关闭、客户端崩溃等异常。在这个情况下，任何一端死掉后， SSH 可以立刻知道，而不会有僵尸程序的发生！

#UseLogin no
##UseLogin 设置是否在交互式会话的登录过程中使用。默认值是"no"。
###如果开启此指令，那么X11Forwarding 将会被禁止，因为login不知道如何处理 xauth cookies 。
####需要注意的是，在SSH底下本来就不接受 login 这个程序的登入，如果指UsePrivilegeSeparation ，那么它将在认证完成后被禁用。
UserLogin no　　　　　　　

#UsePrivilegeSeparation yes
##UsePrivilegeSeparation 设置使用者的权限
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#ShowPatchLevel no

#UseDNS yes
##UseDNS是否使用dns反向解析

#PidFile /var/run/sshd.pid

#MaxStartups 10
##MaxStartups 设置同时允许几个尚未登入的联机，当用户连上ssh但并未输入密码即为所谓的联机，
###在这个联机中，为了保护主机，所以需要设置最大值，预设为10个，而已经建立联机的不计算入内，
####所以一般5个即可，这个设置可以防止恶意对服务器进行连接

#MaxAuthTries 6
##MaxAuthTries 用来设置最大失败尝试登陆次数为6，合理设置辞职，可以防止攻击者穷举登录服务器
#PermitTunnel no

 

############3.7、开放禁止用户设定############

#AllowUsers<用户名1> <用户名2> <用户名3> ...
##指定允许通过远程访问的用户，多个用户以空格隔开

#AllowGroups<组名1> <组名2> <组名3> ...
##指定允许通过远程访问的组，多个组以空格隔开。当多个用户需要通过ssh登录系统时，可将所有用户加入一个组中。

#DenyUsers<用户名1> <用户名2> <用户名3> ...
##指定禁止通过远程访问的用户，多个用户以空格隔开

#DenyGroups<组名1> <组名2> <组名3> ...
##指定禁止通过远程访问的组，多个组以空格隔开。

# no default banner path
#Banner /some/path

# override default of no subsystems
Subsystem       sftp    /usr/libexec/openssh/sftp-server
ClientAliveInterval 3600
ClientAliveCountMax 0
```

## 6.SFTP是什么

SFTP是Secure FileTransferProtocol的缩写，安全文件传送协议。 

SFTP和FTP是两种协议，它们是不同的，sftp是ssh内含的协议，只要sshd服务器启动了，它就可用，它本身没有单独的守护进程，更不需要ftp服务器启动。

SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多，如果您对网络安全性要求更高时，可以使用SFTP代替FTP。

## 7.SFTP登陆

使用sftp登陆远程服务器，可以

```shell
sftp root@192.168.0.1
```

也可以指定端口号

```shell
sftp -oPort=22 root@192.168.0.1
```

## 8.使用SFTP进行文件上传下载

**下载**

语法（[]为可选参数）

```shell
get [-afPpRr] remote [local] 
```

下载远程文件到本地目录

```shell
get /tmp/test.c ~/
```

下载远程文件夹到本地目录

```shell
get -r /tmp/test.c ~/
```

**上传**

语法（[]为可选参数）

```shell
put [-afPpRr] local [remote]
```

上传本地文件到远程文件夹

```shell
put ~/test.c /tmp/
```

上传本地文件夹到远程目录（会上传本地文件夹下的所有文件）

```shell
put -r ~/test /tmp/
```

## 9.更多SFTP命令

输入 help 或 ? 命令可以查看sftp支持的命令操作：

```java
sftp> help
Available commands:
bye                                退出sftp
exit                               退出sftp
quit                               退出sftp
cd path                            将远程目录更改为'path'
chgrp grp path                     将文件'path'的组更改为'grp'
chmod mode path                    将文件'path'的权限更改为'mode'
chown own path                     将文件'path'的所有者更改为'own'
df [-hi] [path]                    显示当前目录的统计信息或包含'path'的文件系统
get [-afPpRr] remote [local]       下载文件
reget [-fPpRr] remote [local]      恢复下载文件
reput [-fPpRr] [local] remote      恢复上传文件
help                               显示此帮助文本
lcd path                           将本地目录更改为'path'
lls [ls-options [path]]            显示本地目录列表
lmkdir path                        创建本地目录
ln [-s] oldpath newpath            链接远程文件（-s用于符号链接）
lpwd                               打印本地工作目录
ls [-1afhlnrSt] [path]             显示远程目录列表
lumask umask                       将本地umask设置为'umask'
mkdir path                         创建远程目录
progress                           切换进度表的显示
put [-afPpRr] local [remote]       上传文件
pwd                                显示远程工作目录
rename oldpath newpath             重命名远程文件
rm path                            删除远程文件
rmdir path                         删除远程目录
symlink oldpath newpath            符号链接远程文件
version                            显示SFTP版本
!command                           在本地shell中执行'command'
!                                  转到本地shell，输入exit可退出并返回到sftp
?                                  显示此帮助文本
```

**执行本地命令**

如果我们想在进入sftp会话环境下执行本地命令怎么办？只需要在本地命令之前加叹号!即可，示例如下：

```shell
!ls
```

当然，你可以输入 `!` 命令转为本地shell会话，退出本地会话输入 `exit` 即可返回到原sftp会话。


**删除文件和目录**

删除远程文件

```shell
rm path
```

删除远程目录（rmdir只能删除空目录）

```shell
rmdir path  
```

注意：不可以设置参数，如 -rf。

**退出会话**

无论是在 ssh 还是 sftp，都可以使用 `exit` 退出当前会话，sftp 还可以使用 `quit` 、`bye` 命令退出。

## 10.结语

相信本文足以可以让你解决使用终端过程中碰到的绝大多数问题了。

> 参考：https://www.cnblogs.com/ftl1012/p/ssh.html