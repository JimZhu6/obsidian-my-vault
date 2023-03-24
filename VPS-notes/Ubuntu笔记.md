

## Ubuntu 服务器初始化

### Ubuntu16.04升级至18.04

#### 首先更新APT源和软件包至最新

```bash
sudo apt update && sudo apt dist-upgrade && sudo apt autoremove
```

#### 安装和配置Ubuntu update manager

更新完组件后，运行以下命令安装update-manager-core

```bash
sudo apt install update-manager-core
```

打开update-manager配置文件

```bash
sudo nano /etc/update-manager/release-upgrades
```

确保设置为**Prompt=lts**

#### 执行升级命令

```bash
sudo do-release-upgrade -d
```

出现升级提示时，全部选择y

等待所有的软件包下载...安装...到重启... 

当所有操作执行完毕后，系统就升级到最新的Ubuntu 18.04 LTS版本了。



### ubuntu18.04创建、删除用户以及修改权限

#### 创建用户

```bash
# 命令一：这种命令会在登录界面显示用户名
sudo useradd -m XXX -d /home/XXX -s /bin/bash

# 命令二：这种命令会在登录界面隐藏用户名
sudo useradd -r -m -s /bin/bash XXX  # XX指代创建的用户名
```

useradd命令参数意义：
-r：建立系统账号
-m：自动建立用户的登入目录
-s：指定用户登入后所使用的shell

#### 设置密码

```bash
# XXX指创建的用户名
sudo passwd XXX
```

#### 修改用户权限

采用修改系统中/etc/sudoers文件的方法分配用户权限。因为此文件只有r权限，在改动前需要增加w权限，改动后，再去掉w权限。

```bash
sudo chmod +w /etc/sudoers
sudo vim /etc/sudoers
```

然后找到以下代码：

```bash
# User privilege specification
root　ALL=(ALL:ALL) ALL
```

并添加需要sudo权限的用户名：

```bash
# User privilege specification
root　ALL=(ALL:ALL) ALL
# 这一行为添加的代码，XXX表示需要添加权限的用户名
XXX ALL=(ALL:ALL) ALL
```

将sudoers文件的操作权限改为只读模式.

```bash
sudo chmod -w /etc/sudoers
# 刷新权限
systemctl restart sshd
```

##### 禁root登录

```bash
# 找到sshd_config文件
whereis ssh

# 获取最高权限
chmod 777 /etc/ssh/sshd_config

# 编辑文件 找到PermitRootLogin yes一行 将yes修改为no
PermitRootLogin no

# 保存退出
:wq

# 修改权限为只读
chmod 444 /etc/ssh/sshd_config

# 刷新权限
systemctl restart sshd
# 此时root用户已经无法登录
```

#### 删除用户

```bash
# XXX为需要删除的用户名
sudo userdel XXX

# XXX为需要删除的用户名
sudo rm -rf /home/XXX	
```

**删除用户权限相关配置.**

删除或者注释掉`/etc/sudoers`中关于要删除用户的配置，否则无法再次创建同名用户.

##### 删除用户残余信息

删除/home目录下的文件.

```bash
cd /home
# XXX为需要删除的用户名
rm -rf XXX
```

删除/etc/passwd下的用户.

```bash
cat /etc/passwd	
# 此命令是查看系统中的所有用户，找到最后一行，可以发现刚刚创建的用户，再使用vi编辑器删除最后一行。
```

删除/etc/group下的用户组文件.

```bash
cat /etc/group
# 此命令是查看系统中的所有用户组，找到最后一行，可以发现刚刚创建的用户，再使用vi编辑器删除最后一行。
```

删除/var/spool/mail下的邮箱文件.

```bash
cd /var/spool/mail
# XXX为需要删除的用户信息
rm -rf XXX
```



### 使用screen后台化进程常用名命令

常用的几个命令： 

```bash
# 启动一个名字为name的screen
screen -S name 

# 删除某个session
screen -S name -X quit

# 列出所有的screen 
screen -ls

# 就可以回到某个screen了（如不行先detached： screen -d name） 
screen -r name或者id

# ctrl + a + d 可以回到前一个screen，当时在当前screen运行的程序不会停止
```



### 使用htop代替top（进程管理器）

```bash
# 安装htop
sudo apt install htop iftop -y
```

### 设置终端走代理

```bash
# 添加代理
export http_proxy='http://proxyAddress:port'
export https_proxy='http://proxyAddress:port'
```

```bash
# 查看代理
env |grep -i proxy
```

```bash
# 清除代理
unset http_proxy
unset https_proxy
```