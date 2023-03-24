# Raspberry-Openwrt



## 准备工作

推荐使用[openfans的镜像](https://github.com/openfans-community-offical/Debian-Pi-Aarch64)。推荐刷入 无桌面增强版，这个版本是自带docker。

刷入镜像并连接到树莓派后，准备工作就完成了。

## 实现步骤

1. 打开网卡混杂模式

   ```shell
   sudo ip link set eth0 promisc on
   ```

2. 创建docker网络

   输入`sudo ifconfig`获取树莓派的ip地址，使用命令创建：

   ```shell
   docker network create -d macvlan --subnet=192.168.254.0/24 --gateway=192.168.254.1 -o parent=eth0 macnet
   ```

   上面的两个ip地址需要根据自身实际情况做修改。关于macvlan，可以查看[这篇文章](https://www.jianshu.com/p/2b8b6c738bf6)

3. 拉取镜像

   ```shell
   docker pull harryzhang6/openwrt:latest
   ```

4. 创建并启动容器

   ```shell
   docker run --restart always --name openwrt -d --network macnet --privileged harryzhang6/openwrt:latest /sbin/init
   ```

   - `--restart always`参数表示容器退出时始终重启，使服务尽量保持始终可用；

   - `--name openwrt`参数定义了容器的名称；

   - `-d`参数定义使容器运行在 Daemon 模式；

   - `--network macnet`参数定义将容器加入 `maxnet`网络；

   - `--privileged`参数定义容器运行在特权模式下；

   这时可以输入`docker qs`确认容器是否成功运行

5. 进入容器并修改相关参数

   ```shell
   docker exec -it openwrt bash
   ```

   执行此命令后我们便进入 OpenWrt 的命令行界面，首先，我们需要编辑 OpenWrt 的网络配置文件：

   ```shell
   vim /etc/config/network
   ```

   需要修改lan口

   ```
   config interface 'lan'
           option type 'bridge'
           option ifname 'eth0'
           option proto 'static' 
           option ipaddr '192.168.254.100' //需要更改处
           option netmask '255.255.255.0'
           option ip6assign '60'
           option gateway '192.168.254.1' //需要更改处
           option broadcast '192.168.123.255'
           option dns '192.168.254.1' //需要更改处
   ```

   `option gateway`和`option dns`填写**路由器**的 IP，若树莓派获得的 IP 为 `192.168.254.154`，路由器 IP 为`192.168.254.1`。`option ipaddr`项目定义了 OpenWrt 的 IP 地址，在完成网段设置后，IP最后一段可根据自己的爱好修改（前提是符合规则且不和现有已分配 IP 冲突）。

6. 重启网络

   ```shell
   /etc/init.d/network restart
   ```

7. 进入openwrt 管理页面

   输入刚刚设置好的`option ipaddr`,我这里是`192.168.254.100`,就可以看到后台的管理界面。

   用户名：`root`。密码：`password`

