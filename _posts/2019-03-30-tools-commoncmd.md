---
title: 常见的一些命令行
tags: 工具
layout: post
---


总结一些常用的命令。

## Windows

##### 1. 回到跟目录：cd\
##### 2. 到上一层目录：cd..
##### 3. 到其他硬盘(比如D)：d:
##### 4. 查看当前目录下面所有的目录和文件： dir
##### 5. 查看当前目录下面所有的目录和文件，并且按顺序显示： dir /o
##### 6. 创建新目录： mkdir 或者 md folder

## Terminal

##### 1. 删除文件夹：rm -rf folderName 或者 sudo rm -rf folderName
##### 2. 查看node装在哪里： ls -la $(which node)
##### 3. 查看当前目录下面所有的目录和文件： ls

## GIT

##### 1. 克隆和下载项目： git clone  https://github.com/LiMeii/angular-seed-project.git
##### 2. 提交代码： git push -u origin master / git push
##### 3. 设置用户名密码： git config --global user.name "Your Name"  git config --global user.email you@example.com
##### 4. SSL Certificate problem: unable to get local issuer certificate： 解决方案 git config --global http.sslVerify false
##### 5. 到上一层目录：cd..
##### 6. 到其他硬盘(比如D)：d:
##### 7. 查看文件列表: ls
##### 8. 查看隐藏文件：ls -al
##### 9. 查看Git状态：git status / git status -s
##### 10. 查看Git提交历史：git log
##### 11. 添加所有改动文件到暂存区（工作区到暂存区）：git add .
##### 12. 添加某个改动文件到暂存区（工作区到暂存区）：git add 文件名
##### 13. 提交改动文件到版本库（从暂存区到版本库）：git commit -m "comments"
##### 14. 提交改动文件到版本库（直接从工作区到版本库）：git commit -a -m "comments"
##### 15. 工作区和暂存区比较：git diff
##### 16. 工作区和版本库比较：git diff 分支名
##### 17. 暂存区和版本库比较：git diff --cached
##### 18. 撤销所有操作（暂存区覆盖工作区，放弃当前工作区修改内容）：git checkout .
##### 19. 撤销某个文件操作（暂存区覆盖工作区，放弃当前工作区修改内容）：git checkout 文件名
##### 20. 当不小心把当前工作区错误提交到暂存区，回滚到上一个暂存区：git reset HEAD 文件名
##### 21. 查看所有的版本号：git reflog
##### 22. 按版本号回滚版本：git reset --hard 版本号
##### 23. 回退到上一个版本：git reset --hard HEAD^
##### 24. 回退某一个版本的文件到工作区：git checkout 版本号 文件名
##### 25. 查看分支：git branch
##### 26. 创建分支：git branch 分支名
##### 27. 切换分支：git checkout 分支名
##### 28. 创建并切换分支：git checkout -b 分支名
##### 29. 删除分支（需要切换出要删除的分支）：git branch -D 分支名
##### 30. 暂存更改：git stash
##### 31. 还原暂存的内容：git stash pop
##### 32. 合并分支（将指定分支合并到当前所在的分支，需要先切换到当前分支，比如master主分支）：git merge 指定的分支名
##### 33. 把某一个commit合并回主分支（需要先切换到主分支）：git cherry-pick 99daed2(commitID)
##### 34. 推送代码到远程仓库：git push origin master
##### 35. 推送代码到远程仓库（以后再次提交的时候可以省略地址别名和分支名称，直接用命令git push进行提交）：git push -u origin master
##### 36. 拉取最新代码但是不合并代码：git fetch origin master
##### 37. 拉取最新代码并且合并代码：git pull origin master
##### 38. 拉取别人的项目到本地：git clone 项目地址 项目名
##### 39. 初始化Git仓库：git init
##### 40. 查看配置信息：git config --list
##### 41. 创建文件：touch 文件名
##### 42. 移动文件：mv 文件/文件夹 路径
##### 43. 查看文件内容：cat 文件名
##### 44. 删除文件/文件夹：rm 文件名
##### 45. 创建文件夹：mkdir 文件夹名

## Jekyll / Ruby

##### 1. 在windows本地启动项目：bundle exec jekyll server
##### 2. 在mac本地启动项目： jekyll server
##### 3. gem安装某版本的package: gem install rails -v 0.14.1 / gem install rails --version 0.14.1 


    



