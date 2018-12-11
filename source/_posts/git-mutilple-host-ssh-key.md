title: "为多个Git仓库配置SSH Key"
date: 2015-07-15 09:08:16
tags:
- git
- ssh
- github

---

在使用Git做版本控制的时候, 如果使用了远程仓库的话通常有两种连接方式，一种是HTTPS的，另一种是SSH的，
由于远程仓库需要一定的权限才能操作，所以需要认证。如果使用了HTTPS这种方式就需要每一次与远程仓库之间交互都需要输入用户名和密码，而SSH会使用SSH认证方式来连接，只要简单的配置一次即可，不需要每次都输入，所以通常都会选择使用SSH的方式。
使用SSH方式连接的话需要配置SSH Key和配置用户名和密码
<!-- more -->

![git](http://ww4.sinaimg.cn/large/a15b4afegw1eu36wmvgorj20ir01swel)


### 添加SSH Key

1. 生成SSH Key

首先配置用户名和Email,

```
$ git config --global user.name 'your_name'

$ git config --global user.email 'your_mail'

```

把your_name替换成自己的名称，your_mail 替换成你的邮箱，比如mail@liangshuai.me

在Git Bash或者Terminal里面运行

```sh
$ ssh-keygen -t rsa -C "your_mail"
```

your_mail应该与前面的保持一致。
然后会让输入保存key的文件名

```
Generating public/private rsa key pair. 
Enter file in which to save the key (/home/user_name/.ssh/id_rsa):

```

可以输入一个名称来标识，比如

```
id_rsa_home

```

之后一直回车即可在~/.ssh/(~表示用户目录)目录下生成id_rsa_home和id_rsa_home.pub，前者是SSH的私钥，后者是公钥，需要把公钥配置到Git 远程仓库。以GitHub为例，

![ssh-key](http://ww4.sinaimg.cn/large/a15b4afegw1eu38vg831cj20u50mvjv8)


先点击1 处之后点 `Settings` 之后点击3处

在Key里面粘贴id_rsa_home.pub里面的内容，Title填写一个用来标识。

保存之后就可以使用了。

### 多个远程仓库配置SSH Key

如果有同一台电脑需要连接多个远程仓库，比如同时使用Github和Gitlab或者BitBucket,这是就需要为多个远程仓库配置SSH Key，以再添加Gitlab仓库为例

1. 生成Gitlab的SSH

```sh
$ ssh-keygen -t rsa -C "your_gitlab_repo_mail"
```

之后可以输入另一个文件名称

```
id_rsa_gitlab

```

2. 多仓库配置

在~/.ssh/目录里面添加config文件

```sh
$ cd ~/.ssh/
$ touch config
$ nano config

```
配置如下

```
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_home

Host git.gitlab.com
  HostName gitlab.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_gitlab

```
可以使用Git客户端测试一下，如果可以连上，那就OK了，如果有问题，试试下面几步。

之后删除已经缓存的Keys

```sh
$ ssh-add -D

```

可以使用

```sh
$ ssh-add -l
```


查看所有已经添加的Key

如果缺少的话使用如下命令添加


```sh
ssh-add ~/.ssh/id_rsa_company
ssh-add ~/.ssh/id_rsa_home

```

### 为不同的项目配置不同的User

如果需要为不同的项目配置不同的User，可以在项目目录下配置


```sh
$ git config user.name "your_name_1"
$ git config user.email "your_email_2" 

```