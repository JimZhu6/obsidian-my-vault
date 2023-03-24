

> 本文记录了我在腾讯云搭建服务器所写的一些备注点，仅做参考。

## CentOs 7服务器初始化

### 打开联网开关（虚拟机用）

```bash
cd /etc/sysconfig/network-scripts/
ls
vi ifcfg-ens33		(这个文件的名字不是固定的)
```

将文件里的`ONBOOT`改为`yes`，使用`:wq`保存退出

启动网络服务

```bash
service network start 
```

### 安装宝塔面板

```bash
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

安装完成后会显示面板的初始化账号、密码已经登录入口

### 查看内外网ip（虚拟机用）

云服务器用外网、vm虚拟机用内网
查看内网

```bash
ifconfig -a
```

查看外网

```bash
curl ifconfig.me
```

### 打开面板安装依赖

使用ip+登录入口的方式登入面板，第一次打开会让你装依赖，等待安装完成。

### 生成ssh公密钥（用于git）

`cd`回根目录，生成ssh公密钥

```bash
cd
ssh-keygen -t rsa -C "YourEmail@XXX.com"
```

提示`Overwrite(y/n)?`时输入y

连续按三次回车，即生成了公密钥

公密钥位于/root/.ssh/

### 生成多个公密钥

```bash
$ ssh-keygen -t rsa -C 'xxxxx@qq.com' -f ~/.ssh/gitee_id_rsa
$ ssh-keygen -t rsa -C 'xxxxx@qq.com' -f ~/.ssh/github_id_rsa
```

在 ~/.ssh 目录下新建一个config文件，添加如下内容（其中Host和HostName填写git服务器的域名，IdentityFile指定私钥的路径）

```bash
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```

用ssh命令分别测试

```bash
$ ssh -T git@gitee.com
$ ssh -T git@github.com
```

### 共用key

将要共用的公钥私钥复制到当前用户的根目录的.ssh文件夹内：`/.ssh/`

将`id_rsa.pub`复制一份至`authorized_keys`（如果已有`authorized_keys`则将`id_rsa.pub`追加到`authorized_keys`文件里）

需要注意文件的权限：

- authorized_keys	600
- id_rsa.pub	644
- id_rsa	600

修改config文件

```bash
# github
Host github.com
HostName github.com                 # 指向的网站
PreferredAuthentications publickey
IdentityFile /.ssh/github_id_rsa    # 指向的密钥
```



### 安装git

```bash
yum install git		(执行安装)
git --version		(检测版本)
git config --global user.name "***"		(设置全局用户名)
git config --global user.email ***@**.**		(设置全局邮箱)
git config --list		(查看配置)
```

### 克隆git仓库

在宝塔面板的文件里的ssh窗口执行命令

### 安装node

宝塔面板“软件管理”里面有一键安装；

#### 使用EPEL安装

检测是否已安装EPEL

```bash
yum info epel-release		(如果没有输出信息，则执行下面一行的命令)
yum install epel-release
```

安装node

```bash
sudo yum install nodejs
```

卸载node

```bash
yum remove nodejs npm -y
```

手动删除残留

- 进入 /usr/local/lib 删除所有 node 和 node_modules文件夹
- 进入 /usr/local/include 删除所有 node 和 node_modules 文件夹
- 检查 ~ 文件夹里面的"local" "lib" "include" 文件夹，然后删除里面的所有 "node" 和 "node_modules" 文件夹
- 可以使用以下命令查找 `$ find ~/ -name node` `$ find ~/ -name node_modules`

进入 /usr/local/bin 删除 node 的可执行文件

- 删除: /usr/local/bin/npm
- 删除: /usr/local/share/man/man1/node.1
- 删除: /usr/local/lib/dtrace/node.d
- 删除: rm -rf /home/[homedir]/.npm
- 删除: rm -rf /home/root/.npm

#### 通过NVM安装

下载并安装NVM脚本

```bash
curl https://raw.githubusercontent.com/creationix/nvm/v0.13.1/install.sh | bash
source ~/.bash_profile
```

列出所需要的版本

```bash
nvm list-remote
```

安装相应的版本

```bash
nvm install v10.14.1
```

查看已安装的版本

```bash
nvm list
```

切换版本

```bash
nvm use v10.14.0
```

设置默认版本

```bash
nvm alias default v10.14.1
```

卸载指定版本

```bash
nvm uninstall v10.14.1
```

### 宝塔面板ssh连接错误

先在命令行里运行这个命令

```bash
$ pip install paramiko==2.0.2
$　bt reload
```

如果不行的话再执行：

```bash
$ ssh-keygen -q -t rsa -P "" -f /root/.ssh/id_rsa
$ cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys  #可以先单独尝试这一条命令先
$ chmod 600 /root/.ssh/authorized_keys
```



## CentOs 7 搭建shadow socks

[参考地址](https://www.jianshu.com/p/2497dd572958) [参考地址2](http://www.cuittk.cn/2017/10/21/%E8%85%BE%E8%AE%AF%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8B%E5%AE%89%E8%A3%85shadowsocks/) [参考地址3](https://www.vultrcn.com/5.html) 

### 安装组件

```bash
$ yum install m2crypto python-setuptools
$ easy_install pip
$ pip install shadowsocks
```

如果提示m2crypto安装失败，则先安装它的依赖

```bash
$ yum install -y openssl-devel gcc swig python-devel autoconf libtool
```

### 写入配置文件

```bash
$ vi  /etc/shadowsocks.json
```

配置文件如下

```javascript
{
    "server":"my_ip_address",
    "server_port":8888,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"my_password",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

**注意：`my_ip_address`里面的ip地址写的是服务器的内网地址，password是用于连接shadowsocks的密码**

如果要设置多端口，则配置文件如下

```javascript
{
    "server":"my_ip_address",
    "local_address": "127.0.0.1",
    "local_port":1080,
    "port_password": {
         "8888": "my_password",
         "8899": "my_password"
     },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

### 配置防火墙端口

```bash
# 开启端口号8888
$ firewall-cmd --permanent --zone=public --add-port=8888/tcp
$ firewall-cmd --reload

# 无法开启则先打开
$ systemctl start firewalld

# 无法打开则先安装
$ yum install firewalld firewall-config
```

### 启动shadow socks

```bash
# 方法一：前台运行
$ ssserver -c /etc/shadowsocks.json
# 方法二：后台运行
$ nohup ssserver -c /etc/shadowsocks.json &
```

### 关闭后台运行的进程

在执行完`nohup [code] &`后，将会返回一个进程id可以通过`kill`命令结束该进程，如`kill 21199`。

如果忘记了进程id，可以通过`ps`命令查询正在后台运行的进程

```bash
$ ps -ef
# 将会展示所有进程目录，PID是进程id，PPID是父进程id
```

可以通过`grep`命令继续筛选

```bash
$ ps -ef |grep {关键字（进程id|命令）}
```

然后就可以使用`kill`命令关闭进程了

### 杀死所有python进程

```bash
ps -ef | grep python | cut -c 9-15| xargs kill -s 9
```

### 原版 & 魔改版 Google BBR

> 在TCP连接中，由于需要维持连接的可靠性，引入了拥塞控制和流量管理的方法。Google BBR就是谷歌公司提出的一个开源TCP拥塞控制的算法。[详细讲解BBR](https://blog.csdn.net/dog250/article/details/52830576)

#### 注意

1、安装 Google BBR 需升级系统内核，而安装锐速则需降级系统内核，故两者不能同时安装。

2、安装 Google BBR 需升级系统内核，有可能造成系统不稳定，故不建议将其应用在重要的生产环境中。

3、原版和魔改版 Google BBR 在不同地区的服务器上会有不同效果，具体孰优孰劣请分别安装进行测试。

#### 原版BBR

执行命令

```bash
$ wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

中途需要用户按一次回车以继续，安装完成后将会提示重启服务器。

重启后，执行下面的三条命令，对比输出信息是否一致

```bash
$ sysctl net.ipv4.tcp_available_congestion_control
# 输出需为：net.ipv4.tcp_available_congestion_control = reno cubic bbr
```

```bash
$ sysctl net.ipv4.tcp_congestion_control
# 输出需为：net.ipv4.tcp_congestion_control = bbr
```

```bash
$ sysctl net.core.default_qdisc
# 输出需为：net.core.default_qdisc = fq
```

对比过输出一致后，则安装完成

#### 魔改版BBR

```bash
# CentOs 6/7使用这个指令
$ wget --no-check-certificate https://raw.githubusercontent.com/nanqinlang-tcp/tcp_nanqinlang/master/General/CentOS/bash/tcp_nanqinlang-1.3.2.sh && bash tcp_nanqinlang-1.3.2.sh

# Debian 7/8 x64 系统请用这个
$ wget --no-check-certificate https://github.com/nanqinlang-tcp/tcp_nanqinlang/releases/download/3.4.2.1/tcp_nanqinlang-fool-1.3.0.sh && bash tcp_nanqinlang-fool-1.3.0.sh
```

然后系统会自动下载并运行。停顿时会让你选择

```
1.安装内核
2.开启算法
3.检查算法状态
4.卸载算法
```

这时候我们输入`1`开始升级内核。

```bash
Complete!
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.12.10-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-4.12.10-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-735fbcd856b697dbc8d6deb1a11dd712
Found initrd image: /boot/initramfs-0-rescue-735fbcd856b697dbc8d6deb1a11dd712.img
done
kernel-ml-headers-4.12.10-1.el7.elrepo.x86_64
kernel-3.10.0-693.el7.x86_64
kernel-ml-devel-4.12.10-1.el7.elrepo.x86_64
kernel-ml-4.12.10-1.el7.elrepo.x86_64
```

此时输入`reboot`重启服务器。重启后再输入下面的命令

```bash
# CentOS 6/7 x64 系统请用这个
$ bash tcp_nanqinlang-1.3.2.sh

# Debian 7/8 x64 系统请用这个
$ bash tcp_nanqinlang-fool-1.3.0.sh
```

按提示输入2启用算法

直到提示`tcp_nanqinlang is installed!` `tcp_nanqinlang is running!`则为安装完成

## CentOs 7安装ssr

[参考链接](https://ssr.tools/31)

登录root用户，按顺序执行下面三条命令

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh

chmod +x shadowsocks-all.sh

./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

如果在运行第一条命令时提示找不到wget，则运行下面的命令安装

```shell
CentOS：
yum -y install wget
Ubuntu/Debian：
apt-get -y install wget
```

接下来将会让你选择参数

1. 选择安装版本：选择`ShadowsocksR`【2】
2. 设置ssr密码
3. 选择要使用的服务器端口号
4. 选择加密方式
5. 选择协议：建议选择`auth_aes128_md5`开始以下的几种
6. 选择混淆方式
7. 按任意键开始安装

安装完成后重启服务器就好了，配置文件在`/etc/shadowsocks-r/config.json`

### ssr常用命令

```shell
# 启动SSR：
/etc/init.d/shadowsocks-r start
# 退出SSR：
/etc/init.d/shadowsocks-r stop
# 重启SSR：
/etc/init.d/shadowsocks-r restart
# SSR状态：
/etc/init.d/shadowsocks-r status
# 卸载SSR：
./shadowsocks-all.sh uninstall
```

如需修改参数，需修改配置文件`/etc/shadowsocks-r/config.json`



## CentOs 7安装Python3

### 修改软连接的方式升级Python3（不推荐）

[参考链接](https://zhuanlan.zhihu.com/p/33660059)

1. 查看Python版本

   ```bash
   $ python -V
   ```

2. 查看软连接指向

   ```bash
   $ ls -al /usr/bin/python
   ```

3. 重命名软连接

   ```bash
   $ mv /usr/bin/python /usr/bin/python2.7.5
   ```

4. 下载并解压 python

   ```bash
   $ wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tgz
   $ tar -xf Python-3.6.4.tgz 
   ```

5. 安装python

   ```bash
   $ cd Python-3.6.4
   $ ./configure
   $ make
   $ make install
   ```

6. 让系统默认使用Python 3.6.4

   由于软连接指向被修改。此时 yum不能使用。需编辑一下 yum 的配置文件。

   ```bash
   $ vi /usr/bin/yum
   ```

   把文件头部的`#!/usr/bin/python`改成`#!/usr/bin/python2.7.5`。

7. 建立新的链接

   ```bash
   $ rm -rf /usr/bin/python
   $ rm -rf /usr/bin/py
   $ ln -s /usr/local/bin/python3.6  /usr/bin/python
   ```

8. 验证是否成功

   ```bash
   $ python -V
   ```

### 直接安装Python3（两个版本共存，推荐）

直接安装Python3

```bash
$ wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tgz
$ tar -xf Python-3.6.4.tgz 
$ cd Python-3.6.4
$ ./configure
$ make
$ make install
```

然后修改环境变量

```bash
$ ln -s * /usr/bin/python3 #*号修改为Python3的安装路径
```

然后如果想要安装Python3的包，可以使用这条命令

```bash
$ python3 -m pip install *
```

## 在CentOs 7使用pyenv管理Python版本

执行命令，安装pyenv

```bash
$ sudo yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

创建安装目录

```bash
$ mkdir ~/.pyenv
$ git clone git://github.com/yyuu/pyenv.git ~/.pyenv  
```

配置环境变量

```bash
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc  
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc  
$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc  
$ exec $SHELL -l
```

常用命令

```bash
#全局设置Python版本
$ pyenv global 3.5.2

#安装指定版本
$ pyenv install 3.5.2

#查看可用版本
$ pyenv versions

#设置Python版本
$ pyenv local 3.5.2

#设置全局Python版本
$ pyenv global 3.5.2

#设置面向shell的Python版本（unset是取消设置）
$ pyenv shell 3.5.2
$ pyenv shell --unset

#每次安装完依赖包都要执行一次下面的命令
$ pyenv rehash

#卸载指定版本
$ pyenv uninstall 3.5.2

#更新
$ pyenv update
```

**注意事项**：若在安装指定版本的Python时提示`pyenv configure: error: no acceptable C compiler found in $PATH See 'config.log' for more details`时，那就是说明你可能没有安装`gcc`。这时需要执行下面的安装命令

```bash
yum groupinstall "Development Tools"
```



## 安装Python3.7+pipenv

### 安装Python3.7

#### 1.安装依赖包

```bash
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

#### 2. 新建python3存放目录

```bash
mkdir /usr/local/python3
```

#### 3. 安装Python3

注意，python3.7.0 需要安装libffi-devel

```bash
yum install libffi-devel -y
```

#### 4.下载Python3安装包

大家可根据自己需求下载不同版本的Python3，本文下载的是Python3.7.0

解压压缩包，进入解压目录，指定安装目录，安装Python3。

```bash
 wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
 tar -xvJf Python-3.7.0.tar.xz
 cd Python-3.7.0
 ./configure --prefix=/usr/local/python3
 make && make install
```

安装Python3时，会自动安装pip。假如没有，需要自己手动安装。

```bash
yum -y install python-pip
```

#### 5. 创建软链接

```bash
 ln -s /usr/local/python3/bin/python3 /usr/bin/python3
 ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

#### 6. 验证python的不同版本

这时，分别输入`python`和`python3`将会看到两个版本的python共存。

### 安装pipenv

#### 1、安装虚拟环境

```bash
pip3 install --upgrade pip
pip3 install --user --upgrade pipenv
```

#### 2、找到pipenv的路径

```bash
 find / -name "pipenv"    # 应该是会返回下面两条目录，我们选用第一行的目录来创建虚拟链接（选择短的那个？）
>> /usr/local/python3/bin/pipenv
>> /usr/local/python3/lib/python3.7/site-packages/pipenv
```

#### 3、创建虚拟链接

```bash
ln -s /usr/local/python3/bin/pipenv /usr/bin/pipenv
```

#### 4、检查pipenv

```bash
pipenv --version
>> pipenv, version 2018.
```



## 安装Docker

### 一键安装脚本

#### 国外VPS版

```sh
#!/bin/bash
# remove old version
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# remove all docker data 
sudo rm -rf /var/lib/docker

#  preinstall utils 
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# add repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# make cache
sudo yum makecache fast

# install the latest stable version of docker
sudo yum install -y docker-ce

# start deamon and enable auto start when power on
sudo systemctl start docker
sudo systemctl enable docker

# add current user 
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
```



#### 国内VPS版

```sh
#!/bin/bash
# 移除掉旧的版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# 删除所有旧的数据
sudo rm -rf /var/lib/docker

#  安装依赖包
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# 添加源，使用了阿里云镜像
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 配置缓存
sudo yum makecache fast

# 安装最新稳定版本的docker
sudo yum install -y docker-ce

# 配置镜像加速器/阿里云加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF

# 启动docker引擎并设置开机启动
sudo systemctl start docker
sudo systemctl enable docker

# 配置当前用户对docker的执行权限
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
```

**注意**：国内访问docker太慢，一般会配置加速器，此处配置的加速器是163的加速器：`http://hub-mirror.c.163.com`，也可以配置[阿里云的加速器](https://link.zhihu.com/?target=https%3A//blog.csdn.net/kozazyh/article/details/79511723)。



#### 配置阿里云docker加速

登录阿里云管理控制台 -> 容器镜像服务 -> 镜像加速器

获取到你的专属加速地址，填写到上面的`registry-mirrors`字段



## 添加用户

### 添加普通用户

#### 新建用户

需要在**root**用户下执行

```shell
# 新建testuser 用户
adduser testuser
# 给testuser 用户设置密码
passwd testuser
```

#### 建立工作组

```shell
# 新建test工作组
groupadd testgroup
```

#### 新建用户同时增加工作组

```shell
# 新建testuser用户并增加到testgroup工作组
useradd -g testgroup testuser

# 注：：-g 所属组 -d 家目录 -s 所用的shell
```

#### 给已有用户增加工作组

```sh
usermod -G groupname username
```

#### 临时关闭用户

在/etc/shadow文件中属于该用户的行的第二个字段（密码）前面加上`*`就可以了。想恢复该用户，去掉`*`即可

```shell
# 或者使用如下命令关闭用户账号：
passwd testuser –l
# 重新释放：
passwd testuser –u
```

#### 删除用户

```shell
userdel testuser
groupdel testgroup
# 强制删除该用户的主目录和主目录下的所有文件和子目录
usermod –G testgroup testuser
```

#### 显示用户信息

```shell
id user
```



```shell
# 用户列表文件
/etc/passwd
# 用户组列表文件
/etc/group
# 查看系统中有哪些用户
cut -d : -f 1 /etc/passwd
# 查看可以登录系统的用户
cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1

# 查看用户操作
# 查看某一用户
w user
# 查看登录用户
who
# 查看用户登录历史记录
last
```



### 添加用户并替换掉root

#### 添加用户

```shell
# 添加用户
adduser user
# 设置密码
passwd user
```

#### 给用户root权限，并禁止root登录

```shell
# 修改权限，修改文件
chmod +w /etc/sudoers
vim /etc/sudoers

# 修改内容如下
## Allow root to run any commands anywhere
root ALL=(ALL)       ALL
user ALL=(ALL)       ALL （添加这一行）

# 恢复只读权限
chmod -w /etc/sudoers

# 禁止root登录
# 修改权限，编辑文件
chmod 777 /etc/ssh/sshd_config
vim /etc/ssh/sshd_config
# 找到PermitRootLogin yes一行 将yes修改为no
PermitRootLogin no

# 保存退出，修改权限为只读
:wq
chmod 444 /etc/ssh/sshd_config

# 刷新权限
systemctl restart sshd
# 此时root用户已经无法登录
```

