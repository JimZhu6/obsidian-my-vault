## js项目改用ts

> 本文来源：[js进化，迁徙到typescript](https://segmentfault.com/a/1190000009630935)



### 一，首先使用重命名工具`renamex-cli`将项目目录`./src`中的所有js文件后缀 批量改成`.ts`

```js
npm i -g renamex-cli
//then
renamex start -p "src/**/*.js" -r "[name].ts" -t no
```



### 二，根目录新建并配置`tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "es2017",//将编译的.ts文件编译为指定标准js
        "module": "commonjs",//模块规范
        "sourceMap": true, //生成资源映射，以便于调试
        "noEmitHelpers": true,//不生成辅助方法，对应importHelpers
        "importHelpers": true,//引用外部的辅助方法 
        "allowUnreachableCode": true,//允许代码中途return产生无法执行代码
        "lib": ["es2017"],//定义编译时依赖 
        "typeRoots": ["node_modules/@types"],//定义类型定义文件的根目录
        "types": [ 
         //添加新的类型定义库如 @types/lodash 需要在此处定义
        "lodash"
        ],
        "outDir": "./build",//编译输出文件目录，默认等于rootDir
        "rootDir": "./src" //源代码目录在这个目录里编写你的ts文件
    },
    "exclude": [
        "node_modules", //忽略目录
        "**/*.test.ts" //忽略指定类型文件
    ] 
}
```

`compilerOptions -> target` 配置项，表明需要将`ypescript`编译到哪一个js标准
可以根据自己的实际需求配置 `es5|es6|es7...`
如果应用在前端可以改为es5



### 三，安装typescript

1. `npm install --save-dev typescript`
   - 可以在npm run scripts里使用`tsc`命令将`.ts`文件编译为`.js`文件
   - `"tsc": "tsc"` 编译`.ts`文件
   - `"tsc:w": "tsc -w"` 监听`.ts`文件 实时编译
   - 属于开发时依赖放在`devDependencies`配置里

```json
{
    "scripts": { 
        "tsc": "npm run clear && tsc",
        "tsc:w": "npm run clear && tsc -w", 
        "lint": "tslint \"src/**/*.ts\" "
    }
}
```

2. `npm install --save tslib` 从外部引入额外的辅助方法集
- 会在编译后的`.js`文件里自动`require('tslib')`
    - 编译后的代码更美观,不用在每个编译后的`.js`文件都生成辅助方法
    - 减少前端场景中打包体积
    - 属于运行时依赖,无须主动引用,必须放在`dependencies`配置里
    - 需要配置`tsconfig.js -> compilerOptions -> importHelpers:true`



### 四，安装 typescript 类型定义(@types/[package])

- npm install --save-dev @types/node (nodejs环境)
- 其它比如`lodash,react,vue,koa,jquery`都已经有了相关的类型定义库
- 配置类型定义库,需要将`tsconfig.json->compilerOptions->types`添加对应的库名

```json
{
    "compilerOptions": {
        "strictNullChecks": true,
        "moduleResolution": "node",
        "allowSyntheticDefaultImports": true,
        "experimentalDecorators": true, 
        "target": "es6",
        "lib": [
        "dom", //如果是前端环境需要添加此配置
        "es7" //适配es7的语法
        ],
        "types": ["lodash"]
    },
    "exclude": ["node_modules"]
}
// 接下来你就可以在开发工具里看到对应的智能提示了,`lodash`:
```



### 五，为项目中的全局变量创建自定义类型定义文件`globals.d.ts`

```ts
 //globals.d.ts
//应用程序工具库
declare var appUtils: any 
//指向 src/common 的绝对路径
declare var COMMON_PATH: string
//node程序的运行环境状态 development | test | production
declare var NODE_ENV: string

//shims.d.ts 第三方插件变量全局定义 
import * as lodash from 'lodash'
declare global {
    /**
     * lodash
     */
    const _: typeof lodash
}
```