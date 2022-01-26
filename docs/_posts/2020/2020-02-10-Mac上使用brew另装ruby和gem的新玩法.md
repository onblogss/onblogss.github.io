---
layout: post
title: Mac上使用brew另装ruby和gem的新玩法
date: 2020-02-10 11:05:00
author: 薛师兄
tags: Mac
---

众所周知，Mac 开机自带 ruby 环境，位于`/Library/Ruby`系统资源库中，通过终端执行 `ruby -v`可以看到 ruby 的版本号：

```shell
% ruby -v 
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
```

不过平常我们使用 ruby 的时候不建议使用系统自带 ruby 环境，因为会对系统文件目录进行读写，试错成本太大，不适合个人学习使用。那问题来了，在这种情况下，我们如何玩转 ruby 呢？

解决办法：使用 Homebrew 安装 ruby 。

Homebrew 的安装复制官网提供的命令行执行即可：<https://brew.sh>

使用 Homebrew 安装 ruby：

```shell
$ brew install ruby
```

安装成功后，发现ruby的版本还是之前系统默认的：

```shell
% ruby -v 
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
```

不要急，这是因为系统自带的 ruby 环境级别太高，导致我们自行安装的 ruby 环境失效。

### 解决方法（一）

笔者的解决办法是找到 ruby 的安装位置，直接略过环境变量执行 ruby 命令。

找到 Homebrew 的软件安装目录：`/usr/local/Cellar/`

找到 ruby 的安装目录 `/usr/local/Cellar/ruby/2.6.5`

找到 ruby 的命令目录 `/usr/local/Cellar/ruby/2.6.5/bin/`

在使用自行安装的 ruby 时只需要带上目录前缀即可，示例：

```shell
% /usr/local/Cellar/ruby/2.6.5/bin/ruby -v
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin19]
```

使用 gem 的方式：

```shell
% /usr/local/Cellar/ruby/2.6.5/bin/gem -v 
3.0.6
```

假设使用 gem 安装了如 jekyll 的软件，执行 jekyll 命令依旧无效：

```shell
% jekyll --version                                 
zsh: command not found: jekyll
```

同理，只需要找到 jekyll 的命令目录，那如何找到呢？

执行 `gem env`命令可以找到有效的帮助：

```java
% /usr/local/Cellar/ruby/2.6.5/bin/gem env        
RubyGems Environment:
  - RUBYGEMS VERSION: 3.0.6
  - RUBY VERSION: 2.6.5 (2019-10-01 patchlevel 114) [x86_64-darwin19]
  - INSTALLATION DIRECTORY: /usr/local/lib/ruby/gems/2.6.0
  - USER INSTALLATION DIRECTORY: /Users/username/.gem/ruby/2.6.0
  - RUBY EXECUTABLE: /usr/local/opt/ruby/bin/ruby
  - GIT EXECUTABLE: /usr/local/bin/git
  - EXECUTABLE DIRECTORY: /usr/local/lib/ruby/gems/2.6.0/bin
  - SPEC CACHE DIRECTORY: /Users/username/.gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /usr/local/Cellar/ruby/2.6.5/etc
  - RUBYGEMS PLATFORMS:
    - ruby
    - x86_64-darwin-19
  - GEM PATHS:
     - /usr/local/lib/ruby/gems/2.6.0
     - /Users/username/.gem/ruby/2.6.0
     - /usr/local/Cellar/ruby/2.6.5/lib/ruby/gems/2.6.0
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => false
     - :bulk_threshold => 1000
  - REMOTE SOURCES:
     - https://rubygems.org/
  - SHELL PATH:
     - /Library/apache-maven-3.6.1/bin
     - /Library/redis-5.0.5/src
     - /Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home/bin
     - /usr/local/bin
     - /usr/bin
     - /bin
     - /usr/sbin
     - /sbin
     - /usr/local/MacGPG2/bin
```

找到 gem 的执行目录即 EXECUTABLE DIRECTORY 为 ：

```shell
/usr/local/lib/ruby/gems/2.6.0/bin
```

尝试执行 jekyll 命令：

```shell
% /usr/local/lib/ruby/gems/2.6.0/bin/jekyll --version
jekyll 4.0.0
```

惊喜的发现我们已经成功的执行了命令。

但是，如果你使用的是 jekyll 你会发现还是会有一些命令执行异常，因为这些命令的执行过程中依旧使用的环境变量里的 ruby 和 gem。

### 解决方法（二）

将自行安装的 ruby 添加到环境变量中，并保证优先级大于系统自带的 ruby。

系统的环境变量加载顺序为：

```java
/etc/profile
/etc/paths 
~/.bash_profile 
~/.bash_login 
~/.profile 
~/.bashrc
```

依次通过 cat 命令查看，找到 `/etc/paths`中的 `/usr/bin`路径中存放着系统的 ruby 命令执行程序，我们将 brew 安装的 ruby 的 bin 目录插入到`/etc/paths`文件的第一行：

```java
% cat /etc/paths
/usr/local/opt/ruby/bin
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin
```

然后使用 source 命令使之生效：

```java
% source /etc/profile
```

检验 ruby 是否切换成功：

```java
% ruby -v            
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin19]
```

可以看到，ruby 环境已经切换为 brew 安装的。

事情到这里并没有结束，还需要将 gem 安装的软件的执行目录加载到环境变量中，通过执行 `gem env`可以找到 EXECUTABLE DIRECTORY 目录的位置，和上面步骤相同，添加到 `/etc/paths`中：

```java
% cat /etc/paths
/usr/local/opt/ruby/bin
/usr/local/lib/ruby/gems/2.6.0/bin
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin
```

使用 `source`命令使之生效即可。

接下来，你可以放心在自己的 Mac 上使用 ruby 和 gem 了。