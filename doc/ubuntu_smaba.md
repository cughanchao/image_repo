# Ubuntu配置samba文件共享服务

## 1. 安装程序

```bash
sudo apt install samba smbclient samba-common cifs-utils
```

## 2. 配置 samba
打开配置文件：

```bash
sudo vim /etc/samba/smb.conf
```

2.1 如果要共享用户的整个`home`目录，在配置文件最后添加如下内容：

```bash
[homes]
comment = Home Directories
browseable = No
create mask = 0664
directory mask = 0775
read only = No  
valid users = %S
```

2.2 检查配置脚本

执行下面命令，就可以看到去除注释后的完整的配置：
```bash
testparm -s
```


## 3. 新建用户(有用户就不需要新建了)
添加用户
```bash
sudo adduser <usename>
```


## 4. 将用户添加到samba组

```bash
sudo usermod -a -G sambashare <usename>
```


## 5. 给用户设置samba密码(此密码可与用户密码相同也可以不通)

```bash
sudo pdbedit -a -u <usename>
```

## 6. 重启samba服务

```bash
sudo systemctl restart smbd.service  nmbd.service
```


## 7. 挂载服务器共享的目录

1. linux电脑下挂在共享目录

先在终端下执行，确保命令行可以挂载：

```bash
sudo mount.cifs //<ip_addr>/<username> /home/your/dir -o username=<user_name>,password=<you_pass_word>,noperm
```

`/home/your/dir`代表本地挂载点。


验证通过后，可以将配置写入  `/etc/fstab`，实现开机自动挂载。

```bash
//<ip_addr>/<username> /home/your/dir cifs defaults,username=share,password=<you_pass_word> 0 0
```

2. windows下挂在服务器共享目录

打开资源管理器，点击映射网络驱动器，会弹出如下的对话框，填入服务器ip地址和共享的目录名，然后输入服务器的用户名和密码接口。如下图所示。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/cughanchao/test_repo@main/img202104012344576.png)



## 其他命令


查询服务器共享目录，下面的命令可以查询对应ip地址的服务器对外共享的目录名：

```bash
smbclient -L //ip_addr
```