

# git 基本配置



## 1. ssh 生成公钥

github 等代码托管平台都可以通过设置ssh 公钥从而免去输入账号和密码来更新代码。因此需要首先生成本地的ssh 公钥。操作如下。
	
```bash
$ ssh-keygen -t rsa -C "your email"
```

接下来如果之前没有设置过就直接三次回车就好，如果之前设置过，则会提示是否要覆盖，敲Y即可。出现下面的页面就代表你已经设置成功了。 -C选项及后面的email可以不写。
	
	
使用下面的这条指令可以查看生成的秘钥，赋值到github的设置中，就可了。
	
```bash
$ cat ~/.ssh/id_rsa.pub
```

## 2. git 配置查询

```bash
$ git config --list
```

## 3. git 初始用户设置

安装git后，第一次提交代码的时候会要求设置用户名和email，设置如下：

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

设置默认编辑器：

```bash
$ git config --global core.editor vim  # 可以选择自己的编辑器，如 emacs、vs code。
```

## 4. 代理设置

国内直接访问github速度比较慢，如果**有代理服务器**的话可以设置git代理。

Git 目前支持的三种协议 git://、ssh:// 和 http://，其代理配置各不相同：core.gitproxy 用于 git:// 协议，http.proxy 用于 http:// 协议，ssh:// 协议的代理需要配置 ssh 的 ProxyCommand 参数。

a. 设置代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'      # 设置http代理
git config --global https.proxy 'socks5://127.0.0.1:1080'     # 设置https代理
git config --global core.gitproxy 'socks5://127.0.0.1:1080'   # 设置git代理
```

b. 查看代理

```bash
git config --global --get http.proxy
git config --global --get https.proxy
git config --global --get core.gitproxy
```

c. 取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset core.gitproxy
```

d. 使用 ssh 协议的 git 配置代理

```bash
# 在~/.ssh/config 文件后面添加几行，没有可以新建一个
# windows 系统在C:\Users\用户名\.ssh\config，不存在自行创建
Host github.com
User git
# SSH默认端口22， HTTPS默认端口443
Port 22
Hostname %h
# 这里放你的SSH私钥
IdentityFile ~\.ssh\id_rsa@[TOC](这里写自定义目录标题)
# 设置代理, 127.0.0.1:1080 换成你自己代理软件监听的本地地址
# HTTPS使用-H，SOCKS使用-S
ProxyCommand connect -S 127.0.0.1:1080 %h %p
```

## 参考

[1.6 起步 - 初次运行 Git 前的配置](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE)

[8.1 自定义 Git - 配置 Git](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git)

[Git设置代理和取消代理的方式](https://www.cnblogs.com/bien94/p/12491737.html)

[如何为 Git 设置代理？](https://segmentfault.com/q/1010000000118837)

[Git设置代理](https://www.jianshu.com/p/739f139cf13c)

[git使用代理服务器的设置方法](https://freesilo.com/?p=1244)

[[整理]为 git 和 ssh 设置 socks5 协议的代理](https://blog.systemctl.top/2017/2017-09-28_set-proxy-for-git-and-ssh-with-socks5/)

[如何为Git设置代理](http://ghoulich.xninja.org/2020/04/10/how-to-set-proxy-for-git/)