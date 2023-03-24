
## wampserver window 安装 VC 依赖

在 window 安装 wamp 之前，需要保证电脑已经安装了下列 VC 运行库：

- VC9（Microsoft Visual C++ 2008）
- VC10（Microsoft Visual C++ 2010）
- VC11（Microsoft Visual C++ 2012)
- VC13
- PHP7 依赖编译的 VC14（Microsoft Visual C++ 2015）

可以在这里下载：

[最新支持的 Visual C++ 下载](https://support.microsoft.com/zh-cn/help/2977003/the-latest-supported-visual-c-downloads)

[VC13安装包](https://support.microsoft.com/en-us/help/4032938/update-for-visual-c-2013-redistributable-package)



## Windows 校验工具

打开命令行，使用一下命令：

```shell
certutil -hashfile <path to file> MD5
```

命令最后的一个参数可选值：

- MD2
- MD4
- MD5
- SHA1
- SHA256
- SHA384
- SHA512