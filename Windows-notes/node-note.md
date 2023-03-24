

> 本文记录了开发时使用node遇到过的特殊错误事件

## Windows通过npm安装sleep模块

[sleep模块](https://www.npmjs.com/package/sleep)可以让js同步阻塞。

在Windows系统安装的时候，有可能会报错：

```
> sleep@6.1.0 install E:\work\xxx\node_modules\sleep
> node-gyp rebuild


E:\work\xxx\node_modules\sleep>if not defined npm_config_node_gyp (node "C:\Users\Administrator\AppData\Roaming\npm\node_modules\npm\node_modules\npm-lifecycle\node-gyp-bin\\..\..\node_modules\node-gyp\bin\node-gyp.js" rebuild )  else (node "C:\Users\Administrator\AppData\Roaming\npm\node_modules\npm\node_modules\node-gyp\bin\node-gyp.js" rebuild )
gyp ERR! configure error
gyp ERR! stack Error: Command failed: C:\Users\Administrator\AppData\Local\Programs\Python\Python37\python.EXE -c import sys; print "%s.%s.%s" % sys.version_info[:3];
gyp ERR! stack   File "<string>", line 1
gyp ERR! stack     import sys; print "%s.%s.%s" % sys.version_info[:3];
gyp ERR! stack                                ^
gyp ERR! stack SyntaxError: invalid syntax
gyp ERR! stack
gyp ERR! stack     at ChildProcess.exithandler (child_process.js:294:12)
gyp ERR! stack     at ChildProcess.emit (events.js:189:13)
gyp ERR! stack     at maybeClose (internal/child_process.js:970:16)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:259:5)
gyp ERR! System Windows_NT 10.0.16299
gyp ERR! command "C:\\Program Files\\nodejs\\node.exe" "C:\\Users\\Administrator\\AppData\\Roaming\\npm\\node_modules\\npm\\node_modules\\node-gyp\\bin\\node-gyp.js" "rebuild"
gyp ERR! cwd E:\work\xxx\node_modules\sleep
gyp ERR! node -v v10.15.3
gyp ERR! node-gyp -v v3.8.0
gyp ERR! not ok
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! sleep@6.1.0 install: `node-gyp rebuild`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the sleep@6.1.0 install script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
```

这是因为sleep是C++模块，而Windows缺少编译C++的npm模块的依赖，所以需要自己安装：

```
npm install --global --production windows-build-tools
```

这个过程大概需要十分钟。安装完成后，再重新执行`npm install sleep`即可。
