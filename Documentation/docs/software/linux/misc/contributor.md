如何向 Linux 社区贡献代码？
-------------------------

本文用一次真实的补丁提交过程，简单介绍了提交内核补丁到社区的流程，以及补丁的后期处理流程。

#### 前言
Linux 作为世界上最受欢迎的开源操作系统，离不开全世界优秀程序员的代码贡献

每一天都有成千上万的补丁被递交到内核社区，其中有新的特性，也有错误修复等

> Linux 从来都不是一个设计，而是一个演变过程

既然你有这份足以贡献代码的力量，为什么不试试加入到 Linux 的开发中来呢？

**在贡献代码的同时，切记 `be careful`，千万不要提交无用的补丁，浪费维护者的宝贵时间**

一份补丁的处理流程图如下


### 1.配置开发环境

本文所使用的平台信息

* `SoC` - Allwinner F1C200s
* `Linux` - v5.18

这里先提前说明一些问题

1. linux补丁的审阅和合并等工作并不会在github上进行，github上的代码仅作为一份镜像存在

2. 内核开发者通过邮件进行交流，提交的补丁也会在lkml（linux 邮件列表）公开

#### 1.1 配置邮箱环境

非常的建议你配置如下所述的邮件环境，为什么呢？
如果你使用的邮件客户端不是纯文本格式收发邮件，等到后边跟开发者交流的时候，
邮件可能会有html乱码，所以，如果你能保证收发皆为纯文本格式，不附带签名等额外信息，
可以忽略这一步，但为了更好地沟通，请尽量的配置一下。

开发邮箱的结构大致如下：
`mutt` + `esmtp` + `fetchmail` + `procmail`

其中，
`mutt` 负责查阅邮件
`esmtp` 负责邮件的发送
`fetchmail` 从邮件服务器上拉取邮件
`procmail` 负责过滤邮件

配置过程可参考这篇文章：https://www.cnblogs.com/hfwz/p/16226109.html

#### 1.2 编译环境配置

内核编译环境大同小异，此处不再赘述

### 2. 修改代码
---------
```shell
# 修改文件，添加测试打印语句
vim drivers/pinctrl/sunxi/pinctrl-suniv-f1c100s.c

	printk(KERN_DEBUG "I can modify the Linux kernel!\n");
	static int cards_found;
	static int global_quad_port_a; /* global ksp3 port a indication */
	int i, err, pci_using_dac;

```

### 3. 编译测试
---------
make modules

仅编译某一文件夹

### 4. 生成提交并检查
---------

git add 
git commit

#### 4.1 添加git post-commit hook

此`hook`脚本将在`git commit`后自动执行，主要作用为检查patch

```shell
# 如果已存在，则移出备份
vim .git/hook/post-commit

# 添加如下两行内容
#!/bin/sh
exec git show --format=email HEAD | ./scripts/checkpatch.pl --strict --codespell
```

### 5. 生成补丁
---------
```shell
main@main-ThinkPad-X230:~/source/linux$ git format-patch -o /tmp/ HEAD^
/tmp/0001-net-ethernet-intel-fix-debugs-function.patch
```

### 6. 获取补丁对应的maintainer
-------------------------
1. 从patch获取
```shell
main@main-ThinkPad-X230:~/source/linux$ git show HEAD | perl scripts/get_maintainer.pl \
--separator , --nokeywords --nogit --nogit-fallback --norolestats
Jesse Brandeburg <jesse.brandeburg@intel.com>,Tony Nguyen <anthony.l.nguyen@intel.com>,"David S. Miller" 
<davem@davemloft.net>,Eric Dumazet <edumazet@google.com>,Jakub Kicinski <kuba@kernel.org>,Paolo Abeni 
<pabeni@redhat.com>,intel-wired-lan@lists.osuosl.org,netdev@vger.kernel.org,linux-kernel@vger.kernel.org
```

2. 从文件获取
```shell
perl scripts/get_maintainer.pl --separator , --nokeywords --nogit -- \
nogit-fallback --norolestats -f drivers/staging/vt6655/baseband.c
```

### 7. 将补丁发送给maintainer
-----------------------

#### 7.1 通过mutt
```shell
mutt -H /tmp/0001-net-ethernet-intel-fix-debugs-function.patch
# 按提示输入对应信息，收件人可以直接复制上一步获取到的，请保持单行
```

#### 7.2 通过git send-email
配置
```shell
git send-email --annotate HEAD^
```

### 8. 等待答复
---------

### 9. 在邮件中进行交流
-----------------

### 10. maintainer进行审阅，通过递交补丁给核心维护者
---------------------------------------------

### 11. 核心维护者再次检查patch
-------------------------

### 12. Linus将补丁合并到主线代码中
----------------------------

### 13. 相关人员负责将补丁合并到哪些LTS版本
------------------------------------

### 参考资料
[kernelnewbies]()