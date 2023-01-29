# 官网工程化 - gulp 篇

## gulp

gulp 是一种基于流的自动化构建工具，基于 nodeJs 中的 stream（流）来读取和写入数据，相对于 grunt 直接对文件进行 IO 读写来说速度更快。
借助于 gulp，我们可以自动化地完成 js/sass/less/css 等文件的的测试、检查、合并、压缩、格式化，并监听文件在改动后重复指定的这些步骤。

要点：

1. 文件压缩（cssnano uglify-js imgmin）
2. babel 转换
3. 图片转 base64
4. px2rem
5. css 雪碧图
6. 自动添加 css 前缀 autoprefixer

### 组件复用

代码复用：为了代码的可复用性和可维护性（减少需要变动时的修改范围）

插件[gulp-file-include](https://www.npmjs.com/package/gulp-file-include)：a plugin of gulp for file include

#### 用法

教程：[HTML 代码复用实践](https://www.cnblogs.com/villy1029/p/6903612.html)

目录：

    |-node_modules
    |-src
        |-include
        |-index.html
    |-dist
    gulpfile.js

```js:gulpfile.js
var gulp = require('gulp')
var fileinclude = require('gulp-file-include')
gulp.task('fileinclude', function () {
  gulp
    .src('src/index.html')
    .pipe(fileinclude({ prefix: '@@', basepath: '@file' }))
    .pipe(gulp.dest('dist'))
})
```

这里的@file 就是指`src/index.html`，引入到该页面。basepath 就是指公用组件存在的位置。

#### tips

1. 最后构建出来的 include 公用文件会合并到 dist/index.html 里。不会生成单独的文件。
2. src/index.html 引入公共文件时仍需要添加相应的路径：`@@include('include/header.html')`
3. 但公用文件里设置链接时需要考虑 build 出后的相应路径
4. 因为共用的组件是相同的，所以如果组件里有链接，要使用这个组件的页面需要在同一路径下，才不会链接错误。

### css 雪碧图

使用雪碧图：目的是减少网络传输，但设置背景图尺寸大小，感觉很麻烦，而且雪碧图的维护也不怎么便利，好像使用率越来越低了，都被 iconfont 取代了，也可以用构建工具

background-image
background-position

#### iconmoon

> iconmoon 是一个在线工具，可以上传自己的 SVG 格式的图标文件，也可以从其中选择已有的图标， 定制出自己的字体文件。

小图标等用 iconfont 代替：作为单个 DOM 节点使用，可以设置大小、颜色等，非常便利。

个人建议前端来维护这个字体包，每次有新增的图标，让设计师给我们对应的 svg 文件即可，前端自己去 icomoon.io/ 这个网站，导入原来的 selection.json 文件，增量生成新的 css，无比方便。之前，我一直以为 iconfont 只能是单色的呢，其实也可以是多色的，svg 里面多一些 path 而已，设计师会搞定的。生成字体后，前端正常引用即可（引用的时候，多色字体会多一些标签）

使用 base64 格式的图片：有些小图片，可能色彩比较复杂，这个时候再用 iconfont 就有点不合适了，此时可以将其转化为 base64 格式（不能缓存），直接嵌在 src 中，比如 webpack 的 url-loader 设置 limit 参数即可

[Writing efficient CSS selectors](https://csswizardry.com/2011/09/writing-efficient-css-selectors/)

### 私有前缀 css

> 采用 autoprefixer 或者 prefixfree 等，让 CSS 真正的 Write once, run anywhere。

css 带前缀的是浏览器私有实现，不带前缀的是标准，加前缀是为了兼容

PC 端：

不需要在源代码中写私有有属性前缀了，只需要控制你自己需要兼容什么样的浏览器版本。正确的姿势是在项目构建阶段，用`autoprefixer`这个工具来为编译后的 css 自动补全所需的前缀。无论你是用 webpack、gulp、grunt 还是 fis，它都能完美配合。

这个工具中内置了非常详尽的数据，描述每个私有有属性在浏览器下的各版本下，是否需要前缀。配置该工具的时候，只要指明需要兼容的浏览器版本，它就会很智能的按需添加前缀了。如果私有语法与标准有差异，它也能自动处理。

只有一种情况下例外，还是要写前缀，那就是你写的这条 css 属性没有对应的 w3c 标准语法。

移动端：

目前大部分设备的浏览器都是基于 webkit 实现的，虽然其它厂商的也有，但不是主流。所以，移动端可以几乎不用加这些私有前缀，个别由于版本迭代，需要加-webkit-前缀兼容低版本的浏览器就行了。

浏览器版本 browserslist 可以写在 package.json 里
https://github.com/browserslist/browserslist#readme

```json
 "browserslist": [
    "last 1 version",
    "> 1%",
    "maintained node versions",
    "not dead"
  ]
```

```js
autoprefixer({
  browsers: ['last 5 versions', 'Android >= 4.0'],
  cascade: true, // 是否美化属性值 默认：true 像这样：
  // -webkit-transform: rotate(45deg);
  // transform: rotate(45deg);
  remove: true, // 是否去掉不必要的前缀 默认：true
})
```

## gulp 配置

```js: gulpfile.js
const gulp = require('gulp')
const postcss = require('gulp-postcss')
const sourcemaps = require('gulp-sourcemaps')
const autoprefixer = require('autoprefixer') // 添加css前缀
const del = require('del') // 清空目录
const concat = require('gulp-concat') // 文件合并
const connect = require('gulp-connect') // 起server服务
const cssnano = require('gulp-cssnano') // 压缩css
const htmlmin = require('gulp-htmlmin') // 压缩html
const imagemin = require('gulp-imagemin') // 压缩图片
const pngquant = require('imagemin-pngquant') // 压缩png图片
const spritesmith = require('gulp.spritesmith') // 合并sprite图片
const uglify = require('gulp-uglify') // 压缩js
const cache = require('gulp-cache') // 缓存资源
const base64 = require('gulp-base64') // 转换图片
const fileinclude = require('gulp-file-include') // 组件复用
const gutil = require('gulp-util') //排错小工具
const preprocess = require('gulp-preprocess')

/* CLEAN: 清空dist目录
 * ------------------------------------------------------ */
gulp.task('clean', async () => {
  await del(['./dist'])
})

/* HTML: 压缩，开发热更新，生产环境
 * ------------------------------------------------------ */
const htmlMin = () => {
  return gulp
    .src(['./src/*.html', './src/include/*.html'])
    .pipe(
      fileinclude({
        prefix: '@@',
        basepath: '@file',
        indent: true,
      })
    )
    .pipe(
      htmlmin({
        collapseWhitespace: true,
        minifyJS: true,
        minifyCSS: true,
        removeComments: true,
      })
    )
    .pipe(gulp.dest('./dist/'))
}

gulp.task('html:dev', async () => {
  await htmlMin().pipe(connect.reload())
})

gulp.task('html:build', async () => {
  await htmlMin()
})

/* CSS：前缀、转图片为base64、合并重命名、压缩
 * ------------------------------------------------------ */
const cssMin = () => {
  return (
    gulp
      .src('./src/assets/css/*.css')
      .pipe(sourcemaps.init())
      .pipe(postcss([autoprefixer()]))
      // .pipe(sourcemaps.write("."))
      .pipe(
        base64({
          maxImageSize: 8 * 1024, // 只转8kb以下的图片为base64
        })
      )
      .pipe(concat('main.min.css'))
      .pipe(cssnano())
      .pipe(gulp.dest('./dist/assets/css'))
  )
}

gulp.task('css:dev', async () => {
  await cssMin().pipe(connect.reload())
})

gulp.task('css:build', async () => {
  await cssMin()
})

/* JS：合并压缩
 * ------------------------------------------------------ */
const jsMin = () => {
  return gulp
    .src('./src/assets/js/*.js')
    .pipe(preprocess({ context: { NODE_ENV: 'production' } }))
    .pipe(concat('main.min.js'))
    .pipe(
      uglify().on('error', function (err) {
        gutil.log(gutil.colors.red('[Error]'), err.toString())
        this.emit('end')
      })
    )
    .pipe(gulp.dest('./dist/assets/js'))
}

const jsMinDev = () => {
  return gulp
    .src('./src/assets/js/*.js')
    .pipe(preprocess({ context: { NODE_ENV: 'development' } }))
    .pipe(concat('main.min.js'))
    .pipe(
      uglify().on('error', function (err) {
        gutil.log(gutil.colors.red('[Error]'), err.toString())
        this.emit('end')
      })
    )
    .pipe(gulp.dest('./dist/assets/js'))
}

gulp.task('js:dev', async () => {
  await jsMinDev().pipe(connect.reload())
})

gulp.task('js:build', async () => {
  await jsMin()
})

/* IMAGE：压缩
 * ------------------------------------------------------ */
const imageMin = () => {
  return gulp
    .src(['./src/assets/img/*.png', './src/assets/img/*.jpg'])
    .pipe(cache(imagemin({ progressive: true, use: [pngquant()] })))
    .pipe(gulp.dest('./dist/assets/img'))
}

// 合并sprite图
const spriteData = () => {
  return gulp
    .src('./src/assets/img/sprite/*.png')
    .pipe(
      spritesmith({
        imgName: 'img/sprite.png',
        cssName: 'css/sprite.css',
        cssFormat: 'css',
        padding: 10,
      })
    )
    .pipe(gulp.dest('./dist/assets'))
}

gulp.task('image:dev', async () => {
  await imageMin().pipe(connect.reload())
  await spriteData().pipe(connect.reload())
})

gulp.task('image:build', async () => {
  await imageMin()
  await spriteData()
})

/* DEV：开发环境执行对应的开发任务
 * ------------------------------------------------------ */
// 监听源文件变化
gulp.task('watch', () => {
  gulp.watch('./src/assets/css/*.css', gulp.series('css:dev'))
  gulp.watch('./src/assets/js/*.js', gulp.series('js:dev'))
  gulp.watch(['./src/assets/img/*.png', './src/assets/img/*.jpg'], gulp.series('image:dev'))
  gulp.watch('./src/**/*.html', gulp.series('html:dev'))
})

// 启动服务
// visit localhost:3000
gulp.task('server', () => {
  connect.server({
    root: './dist', // 以根目录作为入口
    port: 3000,
    livereload: true,
  })
})

const gulpdev = () => {
  return gulp.series(gulp.parallel('watch', 'server'))
}

gulp.task('dev', gulp.series(gulp.parallel('watch', 'server')))

/* BUILD：生产环境执行对应的构建任务
 * ------------------------------------------------------ */
gulp.task('build', gulp.series('clean', gulp.parallel('html:build', 'js:build', 'image:build', 'css:build')))

/* TEST：测试环境执行对应的构建任务
 * ------------------------------------------------------ */
gulp.task('test', gulp.series('clean', gulp.parallel('html:build', 'js:dev', 'image:build', 'css:build')))

```
