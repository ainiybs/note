**服务器：Ubuntu 14.01 LTS版本**
**客户端：Windows 8.1**

[TOC]

## 需求篇
* 支持git
* 支持ssh

> 注1：虽然git支持多种协议，本次尝试采用ssh协议

> 注2：windows端，git bash自带ssh服务

## 扫盲篇
### git

git就是个分布式系统，自行baidu、google...

### ssh

个人理解的ssh是一种安全协议，其验证过程，大致为：在服务器端保存客户端的公钥，客户端保存自己的私钥。
当客户端向服务器发送连接请求时，将自己的私钥同时发送给服务器，服务器根据私钥计算出客户端的公钥并进行比较，由此验证身份。
纯属个人理解，以后有心情补充。//TODO

## 服务器篇
说白了，服务器篇就两件事情，一是必须有个git仓库，二是，必须拥有客户端的公钥。

### git仓库的创建

其实，在创建仓库之前，做了很多准备工作，看起来都是为安全性考虑的，包括：

1. 新建一个git用户

   > sudo adduser git


2. 各种权限设置，暂时没管，反正就是git用户下的东西，别人不能碰之类的//TODO
3. 系统设置，如不允许通过shell登陆git啥的

上述工作都做好了（其实都不用做的），就可以创建git仓库了。既然我们新增了git的用户，那么就可以在git下创建仓库了。


如果是在别的用户下创建仓库，需要做一个链接。啥命令来着，我也忘了。//TODO

git仓库好像有很多种创建方式，诸如从无到有的创建啊，利用现有的文件创建啊之类的。开始学习嘛，就根据教程学习。我看的教程就是创建一个裸的仓库。

```
su git 
cd ~
mkdir work
cd work
git init --bare sample.git
```

上述命令就是在`/home/git/work`文件夹下面初始化了裸的git仓库。虽然是个裸仓库，但已经可以使用git clone命令进行克隆了。

### 保存客户端的公钥

这个其实很简单，保存地址为`/home/git/.ssh/authorized_keys`，如果没有这个目录或者这个文件，自己创建一个就行，剩下的就是把客户端的公钥加进来了。
注1：后面其实有更牛B的法子，一步到位。

## 客户端篇
客户端其实就干了一件事：生成自己的密钥，让服务器去玩吧。
Windows是怎么整SSH的，不清楚，既然git bash自带ssh服务，不妨使用下。

### 生成密钥

> ssh-keygen -t rsa -C “*****@gmail.com”

命令执行后，大约就会在自己的user文件夹下生成一个.ssh的目录，里面有`id_rsa`和`id_rsa.pub`两个文件，将`id_rsa.pub`里面的内容加到3.2里面提到的文件就可以了。

之后就可以自己找个地方clone玩去了。

## SSH扫盲篇
### 登陆命令
SSH主要用于远程登陆，如果以用户名user登陆远程主机host:

> ssh user@host [ssh nacon@192.168.XX.XX]

默认端口为22，可以在登陆时使用 -p 参数指定端口

> ssh -p 2222 user@host

### 密码登录
如果你是第一次登录对方主机，系统会出现提示：无法确认host主机的真实性，只知道它的公钥指纹，问你还想继续连接吗？
确认链接后，会要求输入密码，这时就可以使用密码登录了

###公钥登陆
所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。
登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。
远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。

生成公钥

> ssh-keygen

将公钥传送到Host

> ssh-copy-id user@host

若执行失败，则查看远程主机的/etc/ssh/sshd_config这个文件，看看下面几行是否被注释
　　RSAAuthentication yes
　　		PubkeyAuthentication yes
　　		AuthorizedKeysFile .ssh/authorized_keys
去掉注释后，重启ssh服务

> service ssh restart

### authorized_keys文件

远程主机将用户的公钥，保存在登录后的用户主目录的`$HOME/.ssh/authorized_keys`文件中。公钥就是一段字符串，只要把它追加在authorized_keys文件的末尾就行了。

这里不使用上面的ssh-copy-id命令，改用下面的命令，解释公钥的保存过程：

> ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

这条命令完成的功能为:

1. "$ ssh user@host"，表示登录远程主机
2. 单引号中的mkdir .ssh && cat >> .ssh/authorized_keys，表示登录后在远程shell上执行的命令;
3. "$ mkdir -p .ssh"的作用是，如果用户主目录中的.ssh目录不存在，就创建一个
4. 'cat >> .ssh/authorized_keys' < /.ssh/id_rsa.pub的作用是，将本地的公钥文件~/.ssh/id_rsa.pub，重定向追加到远程文件authorized_keys的末尾。

写入authorized_keys文件后，公钥登录的设置就完成了。





