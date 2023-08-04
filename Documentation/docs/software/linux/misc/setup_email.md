linux mutt+esmtp+fetchmail+procmail 命令行邮箱开发环境配置

#### 安装如下软件包
```shell
main@main:~# sudo apt install fetchmail procmail mutt esmtp -y
```

#### 参考如下缺省配置文件
1. `~/.fetchmailrc`
这里以foxmail为例，user后面填的是你的邮箱地址，password是你的imap授权码，需要在网页版邮箱设置中获取
```shell
poll imap.qq.com protocol imap port 993 user "************" password "****************" keep ssl

mimedecode
mda "/usr/bin/procmail"
```

2. `~/.procmailrc`
```shell
MAILDIR=$HOME/Mail
# main 是我的用户名
DEFAULT=/var/mail/main
VERBOSE=off
LOGFILE=/tmp/procmaillog

:0:
inbox/
```

3. `~/.muttrc`
```shell
set sendmail="/usr/bin/esmtp"
set envelope_from=yes
set from="发件人名字 发件人邮箱"
set use_from=yes
set edit_headers=yes
#set move=yes
set charset="utf-8"

set mbox_type = Maildir
set folder = "$HOME/Mail"
set spoolfile = "$HOME/Mail/inbox"
set mbox="$HOME/Mail/seen"
set record="$HOME/Mail/sent"
set postponed="$HOME/Mail/draft"
```

4. `~/.esmtprc`
同理，identity和username是邮箱地址, password为授权码
```shell
identity "****************"
hostname smtp.qq.com:587
username "****************"
password "****************"
starttls required
```

#### 拉取邮件
```shell
fetchmail -a
```


#### Mutt 查阅邮件
> 首次打开会创建本地邮箱路径
```shell
mutt
```


#### 参考资料
https://www.jianshu.com/p/6a1020dc62f9