# Linux命令

#### 将输出信息重定向到文件命令

```bash
# 将正常信息输出到文件，报错、警告信息还是会输出到屏幕
sh ***.sh > debug.log

# 将错误信息输出到文件，正常信息还是会输出到屏幕
sh ***.sh 2> debug.log

# 将所有信息输出到文件
sh ***.sh > debug.log 2>&1
```



#### 压缩与解压文件

在Linux系统里常见的压缩文件有下面三种

```bash
*.tar         #    tar程序打包产生的文件
*.tar.gz      #    由tar程序打包并由gzip程序压缩产生的文件
*.tar.bz2     #    由tar程序打包并由bzip2程序压缩产生的文件
```

tar常用参数

```bash
# 注意tar的第一参数必须为命令选项，即不能直接接待处理文件
基本格式：tar [Options] file_archive
　　常用命令参数：
　　# 指定tar进行的操作，以下三个选项不能出现在同一条命令中
　　# 创建一个新的打包文件(archive)
　　-c
　　# 对打包文件(archive)进行解压操作
　　-x
　　# 查看打包文件(archive)的内容,主要是构成打包文件(archive)的文件名
　　-t

# 指定支持的压缩/解压方式，操作取决于前面的参数，若为创建(-c),则进行压缩，若为解压(-x),则进行解压，不加下列参数时，则为单纯的打包操作
　　# 使用gzip进行压缩/解压，一般使用.tar.gz后缀
　　-z
　　# 使用bzip2进行压缩/解压，一般使用.tar.bz2后缀
　　-j

# 指定tar指令使用的文件，若没有压缩操作，则以.tar作为后缀
　　# -f后面接操作使用的文件，用空格隔开，且中间不能有其他参数，推荐放在参数集最后或单独作为参数
　　# 文件作用取决于前面的参数，若为创建(-c),则-f后为创建的文件的名字(路径)，若为(-x/t),则-f后为待解压/查看的打包压缩文件名
　　-f filename

　　# 其他辅助选项
　　# 详细显示正在处理的文件名
　　-v
　　# 将解压文件放置在 -C 指定的目录下
　　-C Dir
　　# 保留文件的权限和属性，在备份文件时较有用
　　-p(小写)
　　# 保留原文件的绝对路径，即不会拿掉文件路径开始的根目录，则在还原时会覆盖对应路径上的内容
　　-P(大写)
　　# 排除不进行打包的文件
　　--exclude=file
```

tar常用指令方式

```bash
# -c为创建一个打包文件，相应的-f后面接创建的文件的名称，使用了.tar.bz2后缀，-j标志使用bzip2压缩，最后面为具体的操作对象/etc目录
压缩： tar -cvjpf etc.tar.bz2 /etc

# -t为查看操作，则-f对应所查看的文件的名称，文件后缀显示使用bzip2进行压缩，所以加入-j选项，-v会显示详细的权限信息
查看：tar -tvjf　etc.tar.bz2

# -x为解压操作，则-f指定的是解压使用的文件，文件后缀显示使用bzip2进行压缩，所以加入-j选项，即使用bzip2解压
# 若只解压指定打包文件中的一个文件，在上述指令的最后加上待解压文件名作为参数即可
解压：tar -xvjf　etc.tar.bz2
```

#### systemctl命令

```bash
$ systemctl is-enabled servicename.service # 查询服务是否开机启动
$ systemctl enable xxx.service # 开机运行服务
$ systemctl disable xxx.service # 取消开机运行
$ systemctl start xxx.service # 启动服务
$ systemctl stop xxx.service # 停止服务
$ systemctl restart xxx.service # 重启服务
$ systemctl reload xxx.service # 重新加载服务配置文件
$ systemctl status xxx.service # 查询服务运行状态
$ systemctl --failed # 显示启动失败的服务
```