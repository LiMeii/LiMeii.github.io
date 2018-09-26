---
title: vscode debug
layout: post
---

# 如何配置chrome和VS Code调试前端代码

<div class="title-meta">
    <span>
    Sep 26, 2018
    </span>
    <span class="title-bullet">•</span>
    <span>
     by <a class="github-link" href="http://github.com/limeii">Mei</a>
    </span>
</div>

最开始用Angular5的时候，开发工具用的是VS code，不知道要怎么用chrome调试typescript的代码，看了VS Code官方文档，才知道具体配置方案。

这篇文章就是教你怎么结合chrome浏览器和VS Code调试前端代码，主要介绍从VS Code里launch chrome浏览器，并且在VS Code里设置断点，然后在chrome操作网页从而hit断点进行调试。

关于更多调试方法，比如attach DevTools到浏览器tab从而进行调试，你可以参考官方文档 [VS Code Debugging](https://code.visualstudio.com/docs/editor/debugging)


#### 第一步在VS Code里装 Debugger for Chrom 小插件



i[install chrome extension](https://limeii.github.io/assets/images/posts/tools/tools-debug-install.png){:height="100%" width="100%"}



```html
1. 点击最左边工具栏中的 Extensions
2. 在左上角搜索里输入 Debugger， 然后在搜索结果中找到 Debugger for Charom 点击 install
```

#### 第二步在你的项目里设置lanuch configuration文件


i[setup launch config](https://limeii.github.io/assets/images/posts/tools/tools-debug-config.png){:height="100%" width="100%"}



```html
1. 在VS Code中打开你的项目
2. 点击最左边工具栏中 Debug
3. 在最左上角选择 launch chrome against localhost
4. 第二步以后会出现右边的luanch.json文件，这个文件会自动建在你的项目根目录下面：project root folder -> .vscode -> lanuch.json
5. 按照你调试的需求配置文件中的属性参数
```


#### 第三步调试

i[debugging](https://limeii.github.io/assets/images/posts/tools/tools-debug-debuging.png){:height="100%" width="100%"}



```html
1. 点击最左边工具栏中 Debug
2. 点击最上角绿色 Start Debugging 按钮
3. 在你的项目中设置断点
4. 也可以watch 你想要看的某个参数的值
5. 在开始debugging以后 右上角有调试按钮
```


常见的调试快捷键有：
- F5： 继续/停止
- F10： 下一步
- F11： 调试进某个function
- Shift + F11： 从某个调试function里出来
- Ctrl + Shift + F5：重启
- Shift + F5：结束


