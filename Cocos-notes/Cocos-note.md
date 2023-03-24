# Cocos2d-js

> [文档](https://docs.cocos.com/cocos2d-x/manual/zh/) [Github](https://github.com/cocos2d/cocos2d-x) [API](https://docs.cocos2d-x.org/api-ref/index.html) [论坛](https://forum.cocos.org/c/cocos2d-x/16)

## 安装控制台

[引擎包下载地址](https://www.cocos.com/cocos2dx) [JDK下载地址](https://www.oracle.com/cn/java/technologies/javase-downloads.html) [Apache Ant](https://ant.apache.org/bindownload.cgi)（如需导入Ant，需要定位到里面的bin文件夹） [Android NDK](https://developer.android.com/ndk/downloads) [SDK](https://www.androiddevtools.cn/)

下载引擎包，执行引擎包内的`setup.py`按提示导入Ant、NDK、SDK。

> 亦可直接安装可视化编辑器 [CocosCreator](https://www.cocos.com/creator/)



## 常用指令

[docs](https://docs.cocos.com/cocos2d-x/manual/zh/editors_and_tools/cocosCLTool.html)

### 项目创建

```sh
cocos new <game name> -p <package identifier> -l <language> -d <location>
```

### 项目编译

```sh
cocos compile -s <path to your project> -p <platform> -m <mode> -o <output directory>
```

### 项目运行

```sh
cocos run -s <path to your project> -p <platform>
```

### 项目发布

```sh
cocos deploy -s <path to your project> -p <platform> -m <mode>
```

### 更多帮助

运行 `cocos new --help`、`cocos compile --help`、`cocos deploy --help`等...可以查看更多帮助信息。



## 工程文件

### frameworks/

包含 Cocos 两个引擎与编译工程文件夹（runtime-src）

### index.html

web 程序入口，定义画布。

### main.js

程序入口。

### project.json

主配置文件。当你创建了一个 js 文件时，需要在这里声明。

### publish/

项目编译后的文件。

### src/resource.js

定义静态资源全局变量，在`main.js`用作预加载。



