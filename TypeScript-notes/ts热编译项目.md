---
title: "TypeScript热编译配置"
categories: "笔记类"
tags: "TypeScript"
date: 2019-1-7 15:17:15
---

## 热编译

项目文件结构

```
project
│  gulpfile.js
│  package-lock.json
│  package.json
│  tsconfig.json
│  
├─dist
│      bundle.js
│      index.html
│      
├─node_modules
│
│
└─src
      greet.ts
      index.html
      main.ts
        
```

npm下包

```
npm install --save-dev browserify commonjs gulp gulp-typescript gulp-util tsify typescript vinyl-source-stream watchify
```

gulpfile.js

```js
var gulp = require("gulp");
var browserify = require("browserify");
var source = require('vinyl-source-stream');
var tsify = require("tsify");
// 热更新
var watchify = require("watchify");
var gutil = require("gulp-util");
var paths = {
  pages: ['src/*.html']
};

gulp.task("copy-js", function () {
  return browserify({
      basedir: '.',
      debug: true,
      entries: ['src/main.ts'],
      cache: {},
      packageCache: {}
    })
    .plugin(tsify)
    .bundle()
    .pipe(source('main.js'))
    .pipe(gulp.dest("dist"));
});

var watchedBrowserify = watchify(browserify({
  basedir: '.',
  debug: true,
  entries: ['src/main.ts'],
  cache: {},
  packageCache: {}
}).plugin(tsify));

gulp.task("copy-html", function () {
  return gulp.src(paths.pages)
    .pipe(gulp.dest("dist"));
});

var bundle = function () {
  return watchedBrowserify
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(gulp.dest("dist"));
}

gulp.task(bundle)


gulp.task('default', gulp.series('copy-html', 'bundle'));
watchedBrowserify.on("update", bundle);
watchedBrowserify.on("log", gutil.log);
```

tsconfig.json

```js
{
    "files": [
        "src/main.ts",
        "src/greet.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}
```

此时，在项目更目录使用`gulp`命令即可执行热编译。