
需要准备的工具[adb](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)

## 解锁

前往官网下载[解锁工具](https://www.asus.com/Phone/ZenFone-6-ZS630KL/HelpDesk_Download/)，使用adb安装解锁工具

```sh
adb install Name_of_unlock_app.apk
```

**注意：解锁会清空设备**

安装完成后，手机打开解锁工具，进行解锁。请在这一步之前备份好数据。



## 加锁

```sh
fastboot.exe oem asus-back
```

**注意：加锁会清空设备**



## 安装TWRP

[参考网站](https://forum.xda-developers.com/zenfone-6-2019/development/recovery-unofficial-twrp-recovery-asus-t3937844)

在帖子底部找到指定版本的twrp镜像并下载

手机关机，然后长按`音量+`和`电源键`进入fastboot模式使用下面命令进入twrp

```sh
fastboot boot twrp-3.3.1-23-zenfone6-Q-mauronofrio.img
```

**注意：每次需要进入twrp都需要这样操作**

这时就可以通过twrp安装你需要安装的包



## 官网固件

https://www.asus.com/Phone/ZenFone-6-ZS630KL/HelpDesk_Download/



## 清除数据

在twrp主菜单选择wipe

- **双清：勾选 Dalvik/Art cache、Cache**
  *双清适用于同一个 ROM 直接升级，刷内核或者补丁包，例如从 V6.2 升级到 V6.3 时，双清可以清理缓存，但是又不至于把用户数据和应用程序清除。*
- **三清：勾选 Dalvik/Art cache、Cache、Data**
  *当你不确定你下载的那个包是否真的可用时，建议选这个，万一下载的 ROM 无法刷入，不至于开不了机（如果你的手机里有另外一个绝对可以刷入的包做保底的话，忽略三清用四清或者两清）。*
- **四清：勾选 Dalvik/Art cache、System、Data、Cache**
  *一般在刷入第三方 ROM 之前都应该进行四清，这样可以避免刷完ROM之后出现无法进入系统，或者在使用某些设置和 APP 时出现问题。四清是数据清除最彻底的方式，该操作会将手机原有的系统和其他数据全部清空。所以在操作前如有必要请先进行相应的数据备份。*