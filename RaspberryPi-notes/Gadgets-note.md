# 树莓派的其他玩法

> 记录了使用树莓派部分安装软件的笔记

## 安装百度云盘下载

项目：[BaiduPCS-Go](https://github.com/iikira/BaiduPCS-Go)

确认树莓派系统是32还是64为，运行命令  `getconf LONG_BIT`  结果为：32

确认树莓派的CPU类型：`uname -a` 结果如下：

Linux wzfpi 4.19.57-v7l+ #1244 SMP Thu Jul 4 18:48:07 BST 2019 armv7l GNU/Linux

由此确定为armv7的CPU，系统为32位，所以我需要下载的是BaiduPCS-Go-v3.6-linux-armv7.zip

下载后解压，通过vncviewer传输到桌面，即/home/pi/Desktop下

修改权限：

```sh
chmod 777 /home/pi/Desktop/BaiduPCS-Go
```

执行 `sudo /home/pi/Desktop/BaiduPCS-Go`或者`"/home/pi/Desktop/BaiduPCS-Go"`





## 使用frp内网穿透

[frp项目地址](https://github.com/fatedier/frp)

在具有公网ip的机器上部署frps，配置文件里设置端口号

```ini
# frps.ini
[common]
bind_port = 7000
```

然后启动`./frps -c ./frps.ini`，推荐使用screen等可以后台运行的程序启动



在树莓派上部署frpc，先使用命令`uname -a`检查架构，根据输出信息前往[releases](https://github.com/fatedier/frp/releases)页面下载对应的包。

配置frpc.ini文件

```ini
# frpc.ini
[common]
# 指向公网服务器ip
server_addr = x.x.x.x
# 需要与frps.ini的bind_port相同
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
# 下面的端口号，是在其他设备登录树莓派需要用到的端口号
remote_port = 6000
```

然后启动`./frpc -c ./frpc.ini`，推荐使用screen等可以后台运行的程序启动

**注意：**公网服务器上的防火墙需要释放`server_port`和`remote_port`，否则无法使用

双端启动完成后，可以在其他设备通过`[server_addr]:[remote_port]`连接树莓派





## 使用systemctl设置开机自启命令

### 配置文件

前往systemctl任务目录里创建新任务文件，这里以frp作为例子

```sh
cd /lib/systemd/system
sudo nano frp.service
```

frp.service:

```ini
[Unit]
Description=frp
 
[Service]
User=pi
Type=simple
ExecStartPre=/bin/sleep 10
ExecStart=/home/pi/Desktop/frp/frpc -c /home/pi/Desktop/frp/frpc.ini
 
[Install]
WantedBy=multi-user.target
```



### 常用命令

- 查看启动日志
  ```sh
   journalctl -f
   # or
   journalctl -xe
  ```

- 设置开机自启动

  ```sh
  systemctl enable frp.service
  ```

- 启动frp服务

  ```sh
  systemctl start frp.service
  ```

- 停止开机自启动

  ```sh
  systemctl disable frp.service
  ```

- 查看服务当前状态

  ```sh
  systemctl status frp.service
  ```

- 重新启动服务

  ```sh
  systemctl restart frp.service
  ```

- 查看所有已启动的服务

  ```sh
  systemctl list-units --type=service
  ```

  

