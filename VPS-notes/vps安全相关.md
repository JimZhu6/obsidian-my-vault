

## VPS安全相关设置

查看试图暴力破解你的vps的root密码的ip地址

```bash
sudo grep "Failed password for root" /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr
```



### 修改ssh端口

修改`/etc/ssh/sshd_config`文件，将其中的Port 22改为随意的端口比如Port 47832，port的取值范围是 0 - 65535(即2的16次方)，0到1024是众所周知的端口（知名端口，常用于系统服务等，例如http服务的端口号是80)。

修改后重启ssh服务，CentOs重启ssh服务的命令是：`service sshd restart`。

### 不要使用简单密码

可以考虑使用密码管理器生成的**丧心病狂**般的复杂密码。

### 禁止使用密码登陆，使用RSA私钥登陆

**这条是最重要最有效的。**
跟之前写的[debian ssh 连接android 通过termux](http://link.zhihu.com/?target=https%3A//www.findhao.net/easycoding/1652)里登陆部分是一样的。rsa的原理也不再赘述，[wiki上的条目](http://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/RSA%25E5%258A%25A0%25E5%25AF%2586%25E6%25BC%2594%25E7%25AE%2597%25E6%25B3%2595)描述的很清楚。
先通过`ssh-keygen -t rsa`生成你的客户端的密钥，包括一个私钥和公钥，用`scp id_rsa.pub root@XX.XX.XX.XX:~/`把公钥拷贝到服务器上，注意，生成私钥的时候，文件名是可以自定义的，且可以再加一层密码，所以建议文件名取自己能识别出哪台机器的名字。然后在服务器上，你的用户目录下，新建.ssh文件夹，并将该文件夹的权限设为600，`chmod 600 .ssh`，并新建一个authorized_keys，这是默认允许的key存储的文件。如果已经存在，则只需要将上传的id_rsa.pub文件内容追加进去即可：`cat id_rsa.pub >> authorized_keys`，如果不存在则新建并改权限为400即可。
然后编辑ssh的配置文件：

```bash
# vi /etc/ssh/sshd_config
RSAAuthentication yes #RSA认证
PubkeyAuthentication yes #开启公钥验证
AuthorizedKeysFile .ssh/authorized_keys #验证文件路径
PasswordAuthentication no #禁止密码认证
PermitEmptyPasswords no #禁止空密码
# 最后保存，重启
/etc/init.d/sshd restart
```

### 禁止root用户登录

你可以新建一个用户来管理，而非直接使用root用户，防止密码被破解。
还是修改/etc/ssh/sshd_config

```bash
PermitRootLogin no
```



### 使用denyhosts

> Denyhosts是一个Linux系统下阻止暴力破解SSH密码的软件，它的原理与DDoS Deflate类似，可以自动拒绝过多次数尝试SSH登录的IP地址，防止互联网上某些机器常年破解密码的行为，也可以防止黑客对SSH密码进行穷举。

[denyhost在GitHub上的地址](https://github.com/denyhosts/denyhosts/releases)

#### 在CentOs7安装denyhosts

安装依赖包ipaddr(如果没有找到包，请使用yum update升级yum)

```bash
$ yum install python-ipaddr -y
```

去仓库下载包，解压安装

```bash
$ tar zxvf DenyHosts-3.1.tar.gz 
$ cd denyhosts
$ python setup.py install
```

##### 修改安装配置

```bash
# 将配置文件copy至/etc文件夹
$ cp -rp denyhosts.conf /etc/

# copy一份daemon-control-dist，方便执行的时候我们修改其中的代码
$ cp daemon-control-dist daemon-control

# 设置权限，禁止别人访问改文件
$ chmod 700 daemon-control

# 修改配置文件
$ vi daemon-control
```

daemon-control

```
###############################################
#### Edit these to suit your configuration ####
###############################################

DENYHOSTS_BIN   = "/usr/bin/denyhosts.py"
DENYHOSTS_LOCK  = "/run/denyhosts.pid"
DENYHOSTS_CFG   = "/etc/denyhosts.conf"
```

1、`DENYHOSTS_BIN`默认路径为`/usr/sbin/denyhosts`，启动时会报错` No such file or directory`。追踪了一下`/usr/bin/denyhosts`，并没有找到这个文件。直接改成`/usr/bin/denyhosts.py`即可。
2、`DENYHOSTS_LOCK`是运行后生成的文件，不用管。
3、`DENYHOSTS_CFG`配置文件，之前我们已经将配置文件copy到/etc文件夹下，所以这个默认路径是对的。

启动时报错：` No such file or directory: ‘/var/log/auth.log’`
追踪到目录下时，并没有发现`auth.log`文件，直接创建一个`auth.log`文件即可。

##### 启动Denyhosts

```bash
$ ./daemon-control start
starting DenyHosts: /usr/bin/env python /usr/bin/denyhosts.py --daemon --config=/etc/denyhosts.conf
```

##### 设置开机自启

```bash
# 进入系统启动目录
$ cd /etc/init.d
# 将启动脚本链接到系统启动目录
$ ln -s /usr/local/denyhosts/denyhosts/daemon-control denyhosts

# 设置开机时运行该服务
$ systemctl enable denyhosts

#　启动服务
$ systemctl start denyhosts

# 查看状态
$ systemctl status denyhosts
```

##### 修改配置项

修改`/etc/denyhosts.conf`文件

```bash
SECURE_LOG = /var/log/secure    #sshd的日志文件
HOSTS_DENY = /etc/hosts.deny   #将阻止IP写入到hosts.deny,所以这个工具只支持 支持tcp wrapper的协议
PURGE_DENY = 4w   #过多久后清除已阻止的IP,即阻断恶意IP的时长  （4周）
BLOCK_SERVICE  = sshd   #阻止服务名
DENY_THRESHOLD_INVALID = 5   #允许无效用户登录失败的次数
DENY_THRESHOLD_VALID = 10   #允许普通有效用户登录失败的次数
DENY_THRESHOLD_ROOT = 1    #允许root登录失败的次数
DENY_THRESHOLD_RESTRICTED = 1    #设定 deny host 写入到该资料夹
WORK_DIR = /var/lib/denyhosts    #将deny的host或ip记录到work_dir中
SUSPICIOUS_LOGIN_REPORT_ALLOWED_HOSTS=YES # 启用邮件提醒
HOSTNAME_LOOKUP=YES    #是否做域名反解
LOCK_FILE = /var/lock/subsys/denyhosts    #将DenyHost启动的pid记录到LOCK_FILE中，已确保服务正确启动，防止同时启动多个服务
```

##### 解除已禁止的主机ip

1. 停止DenyHosts服务：

```bash
$sudo service denyhosts stop
```

1. 在 /etc/hosts.deny 中删除你想取消的主机IP
2. 编辑 DenyHosts 工作目录的所有文件，通过

```bash
$ sudo grep 192.168.1.191 /usr/share/denyhosts/data/*
```

3. 然后一个个删除文件中你想取消的主机IP所在的行： 

*/usr/share/denyhosts/data/hosts

*/usr/share/denyhosts/data/hosts-restricted

*/usr/share/denyhosts/data/hosts-root

*/usr/share/denyhosts/data/hosts-valid

*/usr/share/denyhosts/data/users-hosts

4. 添加你想允许的主机IP地址到 /var/lib/denyhosts/allowed-hosts

```bash
vi  /usr/share/denyhosts/data/allowed-hostsps

# We mustn't block localhost
127.0.0.1
192.168.1.*
```

5. 启动DenyHosts服务： `service denyhosts start`