---
layout: post
title: "Vagrantでgulp+BrowserSyncを使いライブリロード"
date: 2015-06-27 15:09:08 +0900 
comments: true
categories: Node
published: true
---

たまに忘れるのでよく使う作業環境のまとめ。
coffeescript環境用。

## directories

```
app/
coffee/**/*.coffee
public/css/**/*.css
public/js/**/*.js
package.json
gulp.coffee
```

## gulp.coffee

普段は一枚ペインかウィンドウ追加して`gulp watch`して放置してる。

```
gulp   = require 'gulp'
$      = require('gulp-load-plugins')()
bs     = require 'browser-sync'

gulp.task 'default', ['minify']

gulp.task 'watch', ->
  gulp.watch('coffee/**/*.coffee', ['minify'])
  bs
    proxy: "peachvideo.dev"
    files: "app/**/*.*, public/**/*.*"

gulp.task 'minify', ->
  gulp
    .src "src/*.coffee", read: false
    .pipe $.plumber()
    .pipe $.browserify
      transform: ['coffeeify']
      extensions: ['.coffee']
      debug: false
    .pipe $.uglify()
    .pipe $.rename "app.js"
    .pipe gulp.dest './public/js'
```

## package.json

適当に使いそうなのをひと通り入れてある、いらないのも混じってる。

```
{
  "devDependencies": {
    "browser-sync": "^2.5.2",
    "coffee": "^1.0.0",
    "coffee-script": "^1.9.0",
    "coffeeify": "^1.0.0",
    "gulp": "^3.8.11",
    "gulp-browserify": "^0.5.1",
    "gulp-concat": "^2.5.2",
    "gulp-load-plugins": "^0.8.1",
    "gulp-minify-css": "^0.5.0",
    "gulp-plumber": "^0.6.6",
    "gulp-rename": "^1.2.0",
    "gulp-uglify": "^1.1.0",
    "gulp-usemin": "^0.3.11",
    "gulp-watch": "^4.1.1",
    "gulp-webserver": "^0.9.0"
  }
}
```

## IPを把握する

WindowsでNodeを使うためのノウハウとかいらないので素直にVagrant内にLinuxで開発環境を立てる。

把握しておくIPだが、Vagrantfile内に記述するIPアドレスでOK。
そのIPに3000ポートでアクセスすればBrowserSync経由でブラウジングできる。
