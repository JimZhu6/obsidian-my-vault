

### Vue DOM元素绑定动态style

```html
:style="{ transform: 'translateY(-' + num1 + 'px)' }"
```

### Vue dom元素绑定多个类名需要以数组的方式书写

```html
:class="[item.colorAndBoxName,
         current1==item.id?'selected1':'',
         current2==item.id?'selected2':'',
         current3==item.id?'selected3':'']"
```

### requestAnimationFrame方法写动画

[RAF参考教程](https://javascript.ruanyifeng.com/htmlapi/requestanimationframe.html#toc0)

```javascript
let animation3Func = function() {
    // do something...
    // 需要执行自调用来执行动画效果
    self.animalId3 = window.requestAnimationFrame(animation3Func);
  };
self.animalId3 = window.requestAnimationFrame(animation3Func);
//清除这个动画效果
window.cancelAnimationFrame(this.animalId3);
```

### Vue使用clipboard（将指定数据存储到剪贴板）

[git地址](https://github.com/zenorocha/clipboard.js)

使用clipboard 2全局注册

```javascript
import VueClipboard from 'vue-clipboard2'
Vue.use(VueClipboard)
```


#### 全局注册时用法

```html
<template>
<div class="container">
    <input type="text" v-model="message">
    <button type="button"
      v-clipboard:copy="message"
      v-clipboard:success="onCopy"
      v-clipboard:error="onError">Copy!</button>
  </div>
</template>

<script>
new Vue({
  el: '#app',
  template: '#t',
  data: function () {
    return {
      message: 'Copy These Text'
    }
  },
  methods: {
    onCopy: function (e) {
      alert('You just copied: ' + e.text)
    },
    onError: function (e) {
      alert('Failed to copy texts')
    }
  }
})
</script>
```



#### 使用clipboard局部组件注册

```html
<span ref="copy"
      data-clipboard-action="copy"
      :data-clipboard-text="arr"
      @click="copy">复制结果</span>
```



```javascript
import clipboard from "clipboard";
export default {
  data(){
    return{
      //动态数组
      arr:[],
      //存储clipboard实例
      copyBtn: null,
      //状态码
      status: 0,
      //待复制内容
      message: ""
    }
  },
    mounted() {
    // 初始化clipboard绑定事件
    //refs里的元素是点击时执行复制操作的按钮
    this.copyBtn = new clipboard(this.$refs.copy);
  },
  methods:{
    copy: function() {
      let _clipboard = this.copyBtn;
      let self = this;
      _clipboard.on("success", function(e) {
        self.status = 1;
        self.message = "你已复制到如下号码:" + e.text;
      });
    }
  },
  watch: {
    status(e) {
      if (e === 1) {
        alert(this.message);
        this.status = 0;
      }
    }
  },
}

```



### Vue脚手架搭建的项目无法热更新解决方法

> 本节来源：[vue项目无法热更新的解决办法](https://note.youdao.com/ynoteshare1/index.html?id=d30efe4bd16a9c084182046c85954876&type=note&from=groupmessage)

修改`vue.config.js`

```js
module.exports = {
  devServer: {
    disableHostCheck: true
  }
}
```



### Vue项目build时移除console.log

> 本节来源：[build时移除console.log()代码](https://note.youdao.com/ynoteshare1/index.html?id=85f391635c44b22275b9f75757c6e2f7&type=note&from=groupmessage)

#### 1. 第一种方法

1. 安装 `babel-plugin-transform-remove-console`
2. 修改 `babel.config.js` 文件

```js
let transformRemoveConsolePlugin = []
if (process.env.NODE_ENV === 'production') {
  transformRemoveConsolePlugin = ['transform-remove-console']
}

module.exports = {
  plugins: [
    ...transformRemoveConsolePlugin
  ]
}
```

#### 2. 第二种方法

1.安装 `terser-webpack-plugin`
2.修改 `vue.config.js` 文件

```js
const TerserPlugin = require('terser-webpack-plugin')

module.exports = {
  configureWebpack: config => {
    config
      .optimization = {
        minimizer: [
          new TerserPlugin({
            terserOptions: {
              compress: {
                drop_console: true
              }
            }
          })
        ]
      }
  }
}
```



### VS Code调试vue代码

> 本节来源：[vscode调试代码](https://note.youdao.com/ynoteshare1/index.html?id=2260a2d0b4954fb48c3fc75d76b8b807&type=note&from=groupmessage)

1. `vscode` 安装 `Debugger for Chrome` 扩展
2. 修改 vue.config.js

```js
module.exports = {
  configureWebpack: {
    devtool: 'source-map'
  }
}
```

1. 按F5 选择 `Chorme` 生成配置文件。
2. 替换配置文件。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "vuejs: chrome",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "breakOnLoad": true,
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/*"
      }
    }
  ]
}
```

1. 如果提示 `Can't find Chrome` ，需要配置 `runtimeExecutable` 为浏览器可执行文件路径。
2. 如果浏览器白屏并且 `vscode` 提示 `Cannot connect to the target`，则需配置 `userDataDir` 为 `true`.



## vue2 Element-UI 按需引入

### 1. 安装插件

```shell
npm install @babel/preset-env -D
```

### 2. 编辑`.babelrc`

```js
{  
    "presets": [["@babel/preset-env", { "modules": false }]],
    "plugins": [
      [
        "component",
        {
          "libraryName": "element-ui",
          "styleLibraryName": "theme-chalk"
        }
      ]
    ]
}
```