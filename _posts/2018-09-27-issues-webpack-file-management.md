---
title: 如何在webpack build结束后移动dist里的文件
tags: 问题
layout: post
---


在 [印度人代码](https://limeii.github.io/2018/09/issues-india-project) 这篇文章中有提到需要在每次webpack build的时候需要每次都动态的去替换MVC中的index.cshtml文件。

## 为什么要替换index.html文件

在现有的这个框架里，每次webpack打包生成的bunlde文件都是 app.bunlde.js / vendor.bundle.js / polyfills.bunlde.js,这个会有 [缓存问题](https://limeii.github.io/2018/09/issues-cache-busting)。

为了解决缓存问题，每次如果对应的bundle文件里面有代码改动，利用webpack contenthash 会生成唯一的hashcode放在bundle文件名里，比如： app.7e90eca5a2a376746ee7.bundle.js.
bundle，文件名不一样，会强制浏览器去拿最新的代码，从而解决缓存问题。

那么在index.cshtml文件里，每次引用的bundle文件也会不一样，所以这个index文件不能像之前一样写死在MVC里面。需要在build结束之后，copy最新的index.cshtml文件去替换MVC框架里对应的文件。

filemanager-webpack-plugin这个插件就可以用来在build结束之后对文件进行处理。

## filemanager-webpack-plugin

我最开始的想法是用copy-webpack-plugin这个插件直接把文件copy到对应的目录下去，实际问题是copy-webpack-plugin这个插件会在index.cshtml文件生成之前执行，导致报错文件找不到。

filemanager-webpack-plugin这个插件就允许你在build之前或者之后 move, delete, 压缩（.zip/.tar/.tar.gz）文件或者文件夹。


## 用法

先安装插件：
```javascript
npm install filemanager-webpack-plugin --save-dev
```
webpack config 文件里：
```javascript
const FileManagerPlugin = require('filemanager-webpack-plugin');

module.exports = {
  plugins: [
    new FileManagerPlugin({
      onEnd: {
        copy: [
          { source: '/path/from', destination: '/path/to' },
          { source: '/path/**/*.js', destination: '/path' },
          { source: '/path/fromfile.txt', destination: '/path/tofile.txt' },
          { source: '/path/**/*.{html,js}', destination: '/path/to' },
          { source: '/path/{file1,file2}.js', destination: '/path/to' },
          { source: '/path/file-[hash].js', destination: '/path/to' }
        ],
        move: [
          { source: '/path/from', destination: '/path/to' },
          { source: '/path/fromfile.txt', destination: '/path/tofile.txt' }
        ],
        delete: [
         '/path/to/file.txt',
         '/path/to/directory/'
        ],
        mkdir: [
         '/path/to/directory/',
         '/another/directory/'
        ],
        archive: [
          { source: '/path/from', destination: '/path/to.zip' },
          { source: '/path/**/*.js', destination: '/path/to.zip' },
          { source: '/path/fromfile.txt', destination: '/path/to.zip' },
          { source: '/path/fromfile.txt', destination: '/path/to.zip', format: 'tar' },
          { 
             source: '/path/fromfile.txt', 
             destination: '/path/to.tar.gz', 
             format: 'tar',
             options: {
               gzip: true,
               gzipOptions: {
                level: 1
               },
               globOptions: {
                nomount: true
               }
             }
           }

        ]
      }
    })
  ],
  ...
}
```

### 具体我项目里用到的copy功能：

```javascript
  plugins: [
     new HtmlWebpackPlugin({
            template: './src/index.cshtml',
            inject: 'body',
            filename: 'index.cshtml'
        }),
        new FileManagerPlugin({
            onEnd: [
                {
                    move: [
                        { source: "./dist/index.cshtml", destination: path.join(process.cwd(), 'Views/Shared/index.cshtml') }
                    ]
                }
            ]
        })
     ]
```