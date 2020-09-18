---
title: 如何配置chrome和VS Code调试前端代码
tags: 工具
layout: post
---


最开始用 Angular5 的时候，开发工具用的是 VS code，不知道要怎么用 chrome 调试 typescript 的代码，看了 VS Code 官方文档，才知道具体配置方案。

这篇文章就是教你怎么结合 chrome 浏览器和 VS Code 调试前端代码，主要介绍从 VS Code 里 launch chrome 浏览器，并且在 VS Code 里设置断点，然后在 chrome 操作网页从而 hit 断点进行调试。

关于更多调试方法，比如 attach DevTools 到浏览器 tab 从而进行调试，你可以参考官方文档 [VS Code Debugging](https://code.visualstudio.com/docs/editor/debugging)


### 第一步，在VS Code里装 Debugger for Chrom 小插件


![install chrome extension](https://limeii.github.io/assets/images/posts/tools/tools-debug-install.png){:height="100%" width="100%"}




- 点击最左边工具栏中的 Extensions
- 在左上角搜索里输入 Debugger， 然后在搜索结果中找到 Debugger for Chrome 点击 install


### 第二步，在你的项目里设置lanuch configuration文件


![setup launch config](https://limeii.github.io/assets/images/posts/tools/tools-debug-config.png){:height="100%" width="100%"}

1. 在 VS Code 中打开你的项目,点击最左边工具栏中 Debug
2. 在最左上角选择 launch chrome against localhost
3. 第二步以后会出现右边的 luanch.json 文件，这个文件会自动建在你的项目根目录下面：project root folder -> .vscode -> lanuch.json
4. 按照你调试的需求配置文件中的属性参数

### 第三步，调试

![debugging](https://limeii.github.io/assets/images/posts/tools/tools-debug-debuging.png){:height="100%" width="100%"}

1. 点击最左边工具栏中 Debug
2. 点击最上角绿色 Start Debugging 按钮
3. 在你的项目中设置断点
4. 也可以 watch 你想要看的某个参数的值
5. 在开始 debugging 以后 右上角有调试按钮

常见的调试快捷键有：
- F5： 继续/停止
- F10： 下一步
- F11： 调试进某个 function
- Shift + F11： 从某个调试 function 里出来
- Ctrl + Shift + F5：重启
- Shift + F5：结束


