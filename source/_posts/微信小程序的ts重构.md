---
title: 微信小程序的ts重构
author: 陈龙
date: 2022-02-18 22:21:16
tags: ['wxml gulp']
categories: ['JavaScript']
---

## 需求背景

之前的微信小程序用的是最原始的 wxml + js 开发的，后决定用使用 ts 进行重构，同时要求尽可能的贴近小程序原生开发

## 技术选择

考虑到要尽可能贴近原生和重构尽量少动 wxml 和 wxcss 代码（时间比较紧迫），决定只对 js 部分进行重构，使用 gulp 将 ts 编译为 js

## 代码实现

首先是`package.json`，里面可以看到打包用到的一些库

```json
{
  "name": "miniprogram",
  "version": "1.0.0",
  "scripts": {
    "dev": "gulp --continue",
    "build": "gulp build",
    "prod": "gulp prod"
  },
  "dependencies": {
    "gulp-rename": "^2.0.0",
    "gulp-shell": "^0.8.0"
  },
  "devDependencies": {
    "del": "^6.0.0",
    "gulp": "^4.0.0",
    "gulp-changed": "^3.2.0",
    "gulp-clean": "^0.4.0",
    "gulp-imagemin": "^5.0.3",
    "gulp-sourcemaps": "^2.6.4",
    "gulp-sync": "^0.1.4",
    "gulp-template": "^5.0.0",
    "gulp-typescript": "^5.0.0-alpha.3",
    "gulp-uglify": "^3.0.2",
    "path": "^0.12.7",
    "typescript": "^4.4.2"
  }
}
```

其次是`gulpfile.js`, 可以看到具体的打包配置

```js
const gulp = require('gulp');
const path = require('path');
const changed = require('gulp-changed');
const clear = require('gulp-clean');
const del = require('del');
const shell = require('gulp-shell');
const uglify = require('gulp-uglify');
const ts = require('gulp-typescript');
const sourcemaps = require('gulp-sourcemaps');
const template = require('gulp-template');
const rename = require('gulp-rename');

const tsProject = ts.createProject('tsconfig.json');

//项目路径
const option = {
  base: 'src',
  allowEmpty: true,
};
const dist = __dirname + '/dist';

// 只编译ts文件，其他文件直接复制到dist文件夹下
const copyPath = ['src/**/!(_)*.*', '!src/**/*.ts'];

const tsPath = ['src/**/*.ts', 'src/app.ts'];

//清空目录
gulp.task('clear', () => {
  return gulp.src(dist, { allowEmpty: true }).pipe(clear());
});

//复制不包含less和图片的文件
gulp.task('copy', () => {
  return gulp.src(copyPath, option).pipe(gulp.dest(dist));
});
//复制不包含less和图片的文件(只改动有变动的文件）
gulp.task('copyChange', () => {
  return gulp.src(copyPath, option).pipe(changed(dist)).pipe(gulp.dest(dist));
});

// 编译
gulp.task('tsCompile', function () {
  return tsProject
    .src()
    .pipe(sourcemaps.init())
    .pipe(tsProject())
    .js.pipe(sourcemaps.write())
    .pipe(gulp.dest('dist'));
});

gulp.task('tsCompileProd', function () {
  return tsProject
    .src()
    .pipe(tsProject())
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});

gulp.task('yarn', shell.task('cd dist && yarn'));

//监听
gulp.task('watch', () => {
  gulp.watch(tsPath, gulp.series('tsCompile'));
  var watcher = gulp.watch(copyPath, gulp.series('copyChange'));
  watcher.on('change', function (event) {
    if (event.type === 'deleted') {
      var filepath = event.path;
      var filePathFromSrc = path.relative(path.resolve('src'), filepath);
      var destFilePath = path.resolve('dist', filePathFromSrc);
      del.sync(destFilePath);
    }
  });
});

const config = (taskName, config) =>
  gulp.task(taskName, function () {
    return gulp
      .src('./config.tmpl')
      .pipe(
        template({
          config: JSON.stringify({
            ...config,
          }),
        })
      )
      .pipe(rename('config.ts'))
      .pipe(gulp.dest('src'));
  });

// 开发测试环境，对应下面的 gulp.task("default")
config('devConfig', { mode: '__DEV__', env: '__TEST__' });

// 线上环境，对应下面的 gulp.task("prod")
config('publishConfig', { mode: '__PROD__', env: '__BETA__' });

//开发并监听, 对应package.json里的scripts的dev指令
gulp.task(
  'default',
  gulp.series(
    'devConfig',
    // sync
    gulp.parallel('copy', 'tsCompile'),
    'yarn',
    'watch'
  )
);

// 线上出包测试，对应package.json里的scripts的prod指令
gulp.task(
  'prod',
  gulp.series(
    // sync
    'clear',
    'publishConfig',
    gulp.parallel(
      // async
      'copy',
      'tsCompileProd'
    ),
    'yarn'
  )
);
```

执行 `yarn dev` 之后，在 `dist` 文件夹内便会生成对应的 `wxml + wxcss + js` 文件， 开发的时候使用微信开发者工具打开 `dist` 文件夹即可
