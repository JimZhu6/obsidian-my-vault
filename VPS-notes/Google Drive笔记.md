# GoogleDrive笔记

## 使用rclone同步或转存文件

在服务器使用[rclone](https://rclone.org/)能很轻松的对谷歌云盘里的文件进行转存或同步操作，其中转存操作包含其他用户共享的文件或文件夹转存到自己的谷歌网盘内。

### 初始化

使用命令安装rclone

```shell
curl https://rclone.org/install.sh | sudo bash
```

安装完成后，输入`rclone config`新建配置

```shell
e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n  # 选择n，新建
name> my_drive  # 输入自定义网盘名称。
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
# ...
13 / Google Drive
   \ "drive"
# ...
Storage> 13  # 选择13，Google Drive
** See help for drive backend at: https://rclone.org/drive/ **

Google Application Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>  # 可留空
Google Application Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>  # 可留空
```

上面两步输入`client_id`及`client_secret`，可以直接按回车跳过，但是不推荐。跳过这个选项程序将使用公用API，导致在高峰时期上传失败。

id以及secret获取方法（教育版账号无法获取此api，忽略本段教程）：

前往[Google API](https://console.developers.google.com/apis/api/drive.googleapis.com/overview)启用API，然后再前往[此页面](https://console.developers.google.com/apis/credentials/oauthclient)创建一个`OAuth 客户端 ID`，应用类型选择`其他`，名称随便填写，接着就会给你id和secret，填到上面的两步输入里面即可。

```shell
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1  # 选1
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id>  # 留空，回车
Service Account Credentials JSON file path
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file>  # 留空，回车
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n  # No
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n  # No
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=XXXXXXXXXXX.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=XXXXXXXXXXXXXXXXXXXX
Log in and authorize rclone for access  # 会弹出浏览器，要求你登录账号进行授权。如果没有弹出，复制上面的链接到浏览器中打开进行授权。
Enter verification code>  # 在这里输入网页上显示的验证码
Configure this as a team drive?
y) Yes
n) No
y/n> y  # “是否是团队盘”根据实际情况选择
Fetching team drive list...
No team drives found in your account--------------------
[Google]
type = drive
scope = drive
token = {"access_token":"XXXXXXXXXXXXXXXXXXXXX"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y  # 设置完成
```

### Linux挂载磁盘相关

#### 挂载磁盘

设置完成后，新建一个你需要挂载的本地目录，这里以/home/gdrive为例

```bash
mkdir -p /root/gdrive
```

挂载为本地磁盘：

```bash
/usr/bin/rclone mount DriveName:Folder LocalFolder \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --transfers 4 \
 --buffer-size 32M \
 --low-level-retries 200
```

这里的`DriveName`是最开始配置时输入的name
`Folder`是Google Drive里的文件夹
`LocalFolder`则是刚才创建的本地目录

挂载成功后，输入`df -h`命令即可看到挂载的磁盘

此时，将文件复制进刚才创建的本地目录中，即可自动上传到Google Drive，Drive中的文件也将自动同步到挂载的磁盘中。

#### 卸载磁盘

使用此命令即可卸载磁盘：

```bash
fusermount -qzu LocalFolder
```

#### 配置开机自动挂载

Rclone默认是不会开机自启并挂载的，我们需要自行设置

```bash
#先将ExecStart后面的指令改成自己的
#再将下面整段命令全部复制到终端一次执行
cat > /etc/systemd/system/rclone.service <<EOF
[Unit]
Description=Rclone
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount DriveName:Folder LocalFolder \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --transfers 4 \
 --buffer-size 32M \
 --low-level-retries 200
Restart=on-abort
User=root

[Install]
WantedBy=default.target
EOF
```

现在就可以用systemctl来启动rclone了：

```bash
systemctl start rclone
```

设置开机启动：

```bash
systemctl enable rclone
```

停止、查看状态可以用：

```bash
systemctl stop rclone
systemctl status rclone
```

### Windows挂载磁盘相关

#### 挂载磁盘

我这里假设我前面添加的名称为my_drive，想要挂载在本机的X:上，并设置缓存目录为F:\Temp（cache路径中请不要带有空格，默认缓存目录为C盘用户目录下， C:\Users\<Your user name>\AppData\Local\rclone）。那么运行以下命令执行挂载（整个my_drive根目录）操作，然后你就会看到一个可爱的X盘出现了~

```sh
rclone mount my_drive:/ x: --cache-dir F:\Temp --vfs-cache-mode writes
```

关于vfs-cache-mode项设置，还是建议看下官方的说明根据自己的需求和网络情况来进行选择 https://rclone.org/commands/rclone_mount/#file-caching 。这里只做简单说明：


>off： In this mode the cache will read directly from the remote and write directly to the remote without caching anything on disk. （本地不做任何缓存，所有文件直接从云端获取并写入。建议网速特别好时（复制粘贴大文件时建议至少100M管以上速度）使用。
>
>minimal： This is very similar to “off” except that files opened for read AND write will be buffered to disks. This means that files opened for write will be a lot more compatible, but uses the minimal disk space. （和off类似，但是已经打开的文件会被缓存到本地。个人推荐，小文件基本够用，但是如果你的网络情况（梯子）不是特别好的话，用writes也行
>
>writes： In this mode files opened for read only are still read directly from the remote, write only and read/write files are buffered to disk first. （如果文件属性为只读则只从云端获取，不然先缓存在本地进行读写操作，随后被同步。个人推荐使用，但是在直接从本地复制文件到my_drive时还是看网络情况
>
>full：In this mode all reads and writes are buffered to and from disk. When a file is opened for read it will be downloaded in its entirety first. （所有的读写操作都会缓存到磁盘中。然后才会同步。不是很推荐。会导致所有文件均被缓存到本地。直到达到你缓存总额（—cache-total-chunk-size，默认大小10G）。但是你网速特别差时也可以使用。

#### 后台运行与开机自启挂载

上面的挂载操作在退出cmd后就自动结束了，所以我们需要让它后台运行。

rclone虽然提供了--daemon参数来实行后台运行，但是该参数并不适合于[windows](https://www.zhiqiang.name/html/tag/windows/)环境中。会有如下提示：

```sh
rclone mount my_drive:/ x: --cache-dir F:\Temp --vfs-cache-mode writes --daemon
2018/05/01 09:54:19 background mode not supported on windows platform
```

所以，我们需要另外想个办法让rclone能够后端运行以及开机自动挂载。

在你之前解压的rclone目录下新建一个文本文件，填入以下内容，请注意修改倒数第二行的WS.Run中相关命令为你上步成功执行的命令，然后将该文件名改为rclone.vbs （后缀名为.vbs即可）

```vb
Option Explicit
Dim WMIService, Process, Processes, Flag, WS
Set WMIService = GetObject("winmgmts:{impersonationlevel=impersonate}!\\.\root\cimv2")
Set Processes = WMIService.ExecQuery("select * from win32_process")
Flag = true
for each Process in Processes
    if strcomp(Process.name, "rclone.exe") = 0 then
        Flag = false
        exit for
    end if
next
Set WMIService = nothing
if Flag then
    Set WS = Wscript.CreateObject("Wscript.Shell")
    WS.Run "rclone mount my_drive:/ x: --cache-dir F:\Temp --vfs-cache-mode writes", 0
end if
```

完成后双击运行，你会看到X盘挂载成功。

补充说明下，如果你看到显示的挂载空间其实是个人空间大小，请参阅此issue: The amount of disk space incorrent when mount Team Drives (gdrive) in Windows 10 · Issue #2288 · ncw/rclone 下载最新的rclone并安装。但超大文件仍建议使用rclone copy或者rclone sync进行复制或者同步操作，而不是直接使用挂载盘，以免卡挂载盘。

**如果你需要中断这个挂载操作，请直接在任务管理器中kill掉rclone.exe进程即可。**

然后将这个文件复制（或者剪贴）到开机项中C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp（Windows 10）即可实现开机自动挂载~



### 常用命令语法

#### rclone命令语法

```shell
# 本地到网盘
rclone [功能选项] <本地路径> <网盘名称:路径> [参数] [参数] ...

# 网盘到本地
rclone [功能选项] <网盘名称:路径> <本地路径> [参数] [参数] ...

# 网盘到网盘
rclone [功能选项] <网盘名称:路径> <网盘名称:路径> [参数] [参数] ...COPY
```

#### 用法示例

```shell
rclone move -v /Download Onedrive:/Download --transfers=1COPY
```

### 常用功能选项

- `rclone copy` - 复制
- `rclone move` - 移动，如果要在移动后删除空源目录，请加上 `--delete-empty-src-dirs` 参数
- `rclone sync` - 同步：将源目录同步到目标目录，只更改目标目录。
- `rclone size` - 查看网盘文件占用大小。
- `rclone delete` - 删除路径下的文件内容。
- `rclone purge` - 删除路径及其所有文件内容。
- `rclone mkdir` - 创建目录。
- `rclone rmdir` - 删除目录。
- `rclone rmdirs` - 删除指定灵境下的空目录。如果加上 `--leave-root` 参数，则不会删除根目录。
- `rclone check` - 检查源和目的地址数据是否匹配。
- `rclone ls` - 列出指定路径下的所有的文件以及文件大小和路径。
- `rclone lsl` - 比上面多一个显示上传时间。
- `rclone lsd` 列出指定路径下的目录
- `rclone lsf` - 列出指定路径下的目录和文件

### 常用参数

> 根据我的个人需要所以会更多的使用Google Drive的配置参数

- `-n` = `--dry-run` - 测试运行，用来查看 rclone 在实际运行中会进行哪些操作。

- `-P` = `--progress` - 显示实时传输进度，500mS 刷新一次，否则默认 1 分钟刷新一次。

- `-v` - 显示当下的内容。

- `--stats Ns` - 每`N`秒显示进度。

- `--drive-skip-gdocs` - 跳过所有列表中的google文档。如果给出的话，gdocs实际上对于rclone变得不可见

- `--drive-server-side-across-configs` - 在两个不同的Google驱动器之间进行服务器端复制时使用这个参数

- `--cache-chunk-size SizeSuffi` - 块的大小，默认5M，理论上是越大上传速度越快，同时占用内存也越多。如果设置得太大，可能会导致进程中断。

- `--cache-chunk-total-size SizeSuffix` - 块可以在本地磁盘上占用的总大小，默认10G。

- `--transfers=N` - 并行文件数，默认为4。在比较小的内存的VPS上建议调小这个参数，比如128M的小鸡上使用建议设置为1。

- `--config string` - 指定配置文件路径，`string`为配置文件路径。

- `--drive-shared-with-me` - 只显示与我共享的文件。

  > This works both with the “list” (lsd, lsl, etc) and the “copy” commands (copy, sync, etc), and with all other commands too.

- `--drive-trashed-only` - 仅显示垃圾桶的文件。

- `--drive-chunk-size` - 设置上传块的大小。

  > 必须为2> = 256k的幂。
  > 使其更大将提高性能，但是请注意，每个块在每次传输时都在内存中进行缓冲。
  > 减少此数量将减少内存使用量，但会降低性能。

- `--ignore-errors` - 跳过错误。比如 OneDrive 在传了某些特殊文件后会提示`Failed to copy: failed to open source object: malwareDetected: Malware detected`，这会导致后续的传输任务被终止掉，此时就可以加上这个参数跳过错误。但需要注意 RCLONE 的退出状态码不会为`0`。

### 日志

rclone 有 4 个级别的日志记录，`ERROR`，`NOTICE`，`INFO` 和 `DEBUG`。默认情况下，rclone 将生成 `ERROR` 和 `NOTICE` 级别消息。

- `-q` - rclone将仅生成 `ERROR` 消息。
- `-v` - rclone将生成 `ERROR`，`NOTICE` 和 `INFO` 消息，**推荐此项**。
- `-vv` - rclone 将生成 `ERROR`，`NOTICE`，`INFO`和 `DEBUG` 消息。
- `--log-level LEVEL` - 标志控制日志级别。

#### 输出日志到文件

使用 `--log-file=FILE` 选项，rclone 会将 `Error`，`Info` 和 `Debug` 消息以及标准错误重定向到 `FILE`，这里的 `FILE` 是你指定的日志文件路径。

另一种方法是使用系统的指向命令，比如：

```shell
rclone sync -v Onedrive:/DRIVEX Gdrive:/DRIVEX > "~/DRIVEX.log" 2>&1COPY
```

### 文件过滤

`--exclude` - 排除文件或目录。

`--include` - 包含文件或目录。

`--filter` - 文件过滤规则，相当于上面两个选项的其它使用方式。包含规则以 `+` 开头，排除规则以 `-` 开头。

#### 文件类型过滤

比如 `--exclude "*.bak"`、`--filter "- *.bak"`，排除所有 `bak` 文件。也可以写作。

比如 `--include "*.{png,jpg}"`、`--filter "+ *.{png,jpg}"`，包含所有 `png` 和 `jpg` 文件，排除其他文件。

`--delete-excluded` 删除排除的文件。需配合过滤参数使用，否则无效。

#### 目录过滤

目录过滤需要在目录名称后面加上 `/`，否则会被当做文件进行匹配。以 `/` 开头只会匹配根目录（指定目录下），否则匹配所目录。这同样适用于文件。

`--exclude ".git/"` 排除所有目录下的`.git` 目录。

`--exclude "/.git/"` 只排除根目录下的`.git` 目录。

`--exclude "{Video,Software}/"` 排除所有目录下的 `Video` 和 `Software` 目录。

`--exclude "/{Video,Software}/"` 只排除根目录下的 `Video` 和 `Software` 目录。

`--include "/{Video,Software}/**"` 仅包含根目录下的 `Video` 和 `Software` 目录的所有内容。

#### 文件大小过滤

默认大小单位为 `kBytes` ，但可以使用 `k` ，`M` 或 `G` 后缀。

`--min-size` 过滤小于指定大小的文件。比如 `--min-size 50` 表示不会传输小于 50k 的文件。

`--max-size` 过滤大于指定大小的文件。比如 `--max-size 1G` 表示不会传输大于 1G 的文件。

> **TIPS:** 博主在实际使用中发现大小过滤两个选项不能同时使用。

#### 过滤规则文件

`--filter-from <规则文件>` 从文件添加包含 / 排除规则。比如 `--filter-from filter-file.txt`。

过滤规则文件示例：

```none
- secret*.jpg
+ *.jpg
+ *.png
+ file2.avi
- /dir/Trash/**
+ /dir/**
- *COPY
```

这里只举例比较常用和简单的一些过滤用法，更复杂和高端的用法可以查看[官方文档](https://p3terx.com/go/aHR0cHM6Ly9yY2xvbmUub3JnL2ZpbHRlcmluZy8=)。

### 环境变量

rclone 中的每个选项都可以通过环境变量设置。环境变量的名称可以通过[长选项名称](https://p3terx.com/go/aHR0cHM6Ly9yY2xvbmUub3JnL2ZsYWdzLw==)进行转换，删除 `--` 前缀，更改 `-` 为`_`，大写并添加前缀 `RCLONE_`。环境变量的优先级会低于命令行选项，即通过命令行追加相应的选项时会覆盖环境变量设定的值。

比如设置最小上传大小 `--min-size 50`，使用环境变量是 `RCLONE_MIN_SIZE=50`。当环境变量设置后，在命令行中使用 `--min-size 100`，那么此时环境变量的值就会被覆盖。

#### 常用环境变量

- `RCLONE_CONFIG` - 自定义配置文件路径
- `RCLONE_CONFIG_PASS` - 若 rclone 进行了加密设置，把此环境变量设置为密码，可自动解密配置文件。
- `RCLONE_RETRIES` - 上传失败重试次数，默认 3 次
- `RCLONE_RETRIES_SLEEP` - 上传失败重试等待时间，默认禁用，单位`s`、`m`、`h`分别代表秒、分钟、小时。
- `CLONE_TRANSFERS` - 并行上传文件数。
- `RCLONE_CACHE_CHUNK_SIZE` - 块的大小，默认5M，理论上是越大上传速度越快，同时占用内存也越多。如果设置得太大，可能会导致进程中断。
- `RCLONE_CACHE_CHUNK_TOTAL_SIZE` - 块可以在本地磁盘上占用的总大小，默认10G。
- `RCLONE_IGNORE_ERRORS=true` - 跳过错误。

