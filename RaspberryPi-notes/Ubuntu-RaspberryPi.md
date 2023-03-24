# Ubuntu安装指南

## 安装

根据Ubuntu[官网介绍](https://ubuntu.com/tutorials/how-to-install-ubuntu-desktop-on-raspberry-pi-4#1-overview)，准备一张SD卡，通过下载Raspberry Pi[专用镜像安装器](https://downloads.raspberrypi.org/imager/imager_latest.exe)刷入指定系统。

## 登录

把烧录好镜像的SD卡插入树莓派，通过网线连接路由器，获取到ip后登录树莓派。默认用户名和密码都是ubuntu，第一次登陆后，需要修改默认密码，修改后重新登录。

## 使用frp做映射

[前往下载](https://github.com/fatedier/frp/releases)最新版本的客户端和服务端二进制文件。frps需部署到有公网的服务器端，frpc部署到内网端。按以下教程配置

### 通过 SSH 访问内网机器

这个示例通过简单配置 TCP 类型的代理让用户访问到内网的服务器。

1. 在具有公网 IP 的机器上部署 frps，修改 frps.ini 文件，这里使用了最简化的配置，设置了 frp 服务器用户接收客户端连接的端口：

   ```ini
   [common]
   bind_port = 7000
   ```

2. 在需要被访问的内网机器上（SSH 服务通常监听在 22 端口）部署 frpc，修改 frpc.ini 文件，假设 frps 所在服务器的公网 IP 为 x.x.x.x：

   ```ini
   [common]
   server_addr = x.x.x.x
   server_port = 7000
   
   [ssh]
   type = tcp
   local_ip = 127.0.0.1
   local_port = 22
   remote_port = 6000
   ```

   `local_ip` 和 `local_port` 配置为本地需要暴露到公网的服务地址和端口。`remote_port` 表示在 frp 服务端监听的端口，访问此端口的流量将会被转发到本地服务对应的端口。

3. 分别启动 frps 和 frpc。

4. 通过 SSH 访问内网机器，假设用户名为 test：

   `ssh -oPort=6000 test@x.x.x.x`

   frp 会将请求 `x.x.x.x:6000` 的流量转发到内网机器的 22 端口。

### 安全地暴露内网服务

这个示例将会创建一个只有自己能访问到的 SSH 服务代理。

对于某些服务来说如果直接暴露于公网上将会存在安全隐患。

使用 `stcp(secret tcp)` 类型的代理可以避免让任何人都能访问到要穿透的服务，但是访问者也需要运行另外一个 frpc 客户端。

1. frps.ini 内容如下：

   ```ini
   [common]
   bind_port = 7000
   ```

2. 在需要暴露到内网的机器上部署 frpc，且配置如下：

   ```ini
   [common]
   server_addr = x.x.x.x
   server_port = 7000
   
   [secret_ssh]
   type = stcp
   # 只有 sk 一致的用户才能访问到此服务
   sk = abcdefg
   local_ip = 127.0.0.1
   local_port = 22
   ```

3. 在想要访问内网服务的机器上也部署 frpc，且配置如下：

   ```ini
   [common]
   server_addr = x.x.x.x
   server_port = 7000
   
   [secret_ssh_visitor]
   type = stcp
   # stcp 的访问者
   role = visitor
   # 要访问的 stcp 代理的名字
   server_name = secret_ssh
   sk = abcdefg
   # 绑定本地端口用于访问 SSH 服务
   bind_addr = 127.0.0.1
   bind_port = 6000
   ```

4. 通过 SSH 访问内网机器，假设用户名为 test：

   `ssh -oPort=6000 test@127.0.0.1`
   
   
   
### 开机自启并在后台运行

在 /usr/lib/systemd/system 下面新建一个文件frps.service，文件内容如下：

```bash
[Unit]
Description=frps daemon

[Service]
Type=simple
#此处把/root/frp_linux_arm64替换成 你的frps的实际安装目录
ExecStart=/home/ubuntu/frp/frpc -c /home/ubuntu/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

- 启动frp `sudo systemctl start frpc`
- 打开自启动 `sudo systemctl enable frpc`
- 重启应用 `sudo systemctl restart frpc`
- 停止应用 `sudo systemctl stop frpc`
- 查看日志 `sudo systemctl status frpc`



## 安装Docker

输入下面命令进行安装

```bash
sudo apt install docker.io
```

查看版本

```bash
docker version
```

启动docker server，需要切换为root用户启动

```bash
service docker start
```

设置开机自启

```bash
systemctl start docker
systemctl enable docker
systemctl start docker.service
systemctl enable docker.service
```

修改源，修改后需要重启服务

```bash
# vi /etc/docker/daemon.json
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

测试docker

```bash
docker pull hello-world
docker run hello-world
```



## 安装docker图形化工具portainer

### 安装

具体安装方法官网就有[介绍](https://www.portainer.io/installation/)。推荐安装[docker](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/)版本，官网介绍基本就2条命令搞定（后面我根据自身需求修改）。

```bash
$ docker volume create portainer_data
$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```



第1句是创建名为portainer_data的数据卷；

第2句复杂点：

> -d 后台运行容器
> -p 8000:8000 -p 9000:9000 做了2个端口映射，将 portainer docker 内的端口8000和9000映射到宿主机的8000和9000端口
> --name=portainer 为容器指定名称portainer
> --restart=always 当 docker 重启时，容器能自动启动
> -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data
> 这里 -v 是路径地址映射，将容器内对应文件映射到宿主机，方便管理
> portainer/portainer-ce这是镜像名称，旧的版本叫做portainer/portainer，应该是不用了

这里本身我就比较奇怪为什么映射2个端口，简单查了一下9000是web管理端口，8000是代理接入端口，一般我们只需要页面管理，我选择只设置9000端口映射。注意-v /var/run/docker.sock:/var/run/docker.sock，这只在 Linux 环境下适用（windows是另外的地址），我的是 Ubuntu 也属于 Linux ，当然也是适用，请不要改动这个路径，它对应 docker 的管理路径，后面的路径 portainer_data，是前面创建的数据卷，可以改动设置到其他文件下，这样数据卷也就没有必要创建了，所以我们修改用一句代码搞定安装 portainer 。

```bash
docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/portainer_data:/data portainer/portainer-ce
```

打开树莓派的ip地址，加端口号9000即可打开管理页面

### 更新

先停止并移除当前版本

```bash
$ docker stop portainer
$ docker rm portainer
```



再拉取新版镜像，新建容器。

```bash
docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/portainer_data:/data portainer/portainer-ce
```

### 

## 挂载U盘

首先把U盘插入树莓派，然后查看一下是否有被识别到。

 ```bash
 sudo fdisk -l
 ```

 这里好像一定要是`sudo`才行，如果不获取权限好像什么都不显示。其中/dev/mmc表示的是TF卡，容量是8G，而/dev/sda/表示的是我们的第一个硬件（U盘），容量也是8G，只有一个分区。

查看了U盘已经正确被识别，现在准备进行挂载。
 新建一个目录

 ```bash
 sudo mkdir /mnt/sda
 ```

 然后挂载设备

 ```bash
 sudo mount -o uid=pi,gid=pi /dev/sda /mnt/sda/
 ```



需要拔出U盘的时候，可以这样取消挂载

 ```bash
 sudo umount /mnt/sda
 ```



如果提示设备在忙，那么可以使用下面的方法尝试。
 问题现象：

```bash
#umount /dev/sda
umount: /mnt/usb: device is busy
```

查找占用目录进程：



```bash
#lsof |grep /mnt/sda
bash 1971 root cwd DIR 8,1 16384 1 /mnt/usb/
bash 2342 root 3r DIR 8,1 16384 1 /mnt/usb/
```

杀掉进程：

```bash
#kill -9 1971
#kill -9 2342
```

卸载：

 ```bash
 #umount /mnt/sda
 ```



### 格式化U盘

首先执行`sudo fdisk -l`查看你的u盘的序号，通常是`/dev/sdb`之类的，U盘分区通常是`/dev/sdb1`

对于u盘我们一般格式化为FAT格式或者FAT32格式，不过在linux下这些会都显示为FAT格式。我们只需要执行命令：
 `sudo mkfs.vfat -F 32 /dev/sdb1`即可将u盘格式化为fat32格式。

```bash
sudo mkfs.ext4 /dev/sda1 # 格式化为ext4分区 
sudo mkfs.ext3 /dev/sda1 # 格式化为ext3分区 
sudo mkfs.ext2 /dev/sda1 #格式化为ext2分区 
```



## 部署photoprism

首先要确保boot里的config文件中`arm_64bit=1`，然后拉取镜像

```bash
docker pull --platform=arm64 photoprism/photoprism:latest
```

启动命令

```bash
docker run -d \
  --name photoprism \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  --restart=always \
  -p 2342:2342 \
  -e PHOTOPRISM_ADMIN_PASSWORD="insecure" \
  -v /photoprism/storage \
  -v ~/mnt/sda/Photoprism:/photoprism/originals \
  photoprism/photoprism
```

[配置选项文档](https://docs.photoprism.app/getting-started/config-options/)

带导入文件夹的启动命令

```bash
docker run -d \
  --name photoprism \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  --restart=always \
  -p 2342:2342 \
  -e PHOTOPRISM_ADMIN_PASSWORD="insecure" \
  -v /photoprism/storage \
  -v ~/Photoprism:/photoprism/originals \
  -v ~/Photoprism/import:/photorism/import \
  photoprism/photoprism

```

