## 前言

Git作为程序员必备的一个版本控制工具，本文跟大家分享一下Git的最基础用法  

**注：本文涉及到的所有命令均在 Git Bash 下执行**

## 配置

设置名称

```
git config --global user.name xxx
```

设置邮箱

```
git config --global user.email xxx@gmail.com
```

## 初始化

创建仓库目录

```
mkdir TestRepo
```
进入仓库目录

```
cd TestRepo
```
初始化仓库

```
git init
```
之后会在根目录下生成.git文件夹

## 设置忽略

在根目录下创建.gitignore文件

```
touch .gitignore
```

添加忽略名单，如忽略README.md文件

```
echo README.md > .gitignore
```
忽略所有txt文件

```
echo *.txt >> .gitignore 
```
将.gitignore文件加入版本管理并提交

## 提交
工作修改之后，添加文件到暂存区

添加单个文件

```
git add README.md
```
添加所有文件

```
git add .
```
从暂存区提交到仓库

提交单个文件

```
git commit -m "commit message" README.md
```

提交所有文件

```
git commit -m "commit message" -a
```
默认提交所有文件

```
git commit -m "commit message"
```
自定义提交信息

```
git commit -s
```
之后打开默认编辑器，编辑提交信息保存后提交

增补提交，不产生新的提交记录

```
git commit --amend
```

## 查看状态

列出被修改的文件

```
git status
```
和暂存区的内容比较

```
git diff
```
和上次提交的内容比较

```
git diff HEAD
```
和上上次提交的内容比较

```
git diff HEAD^
```
和历史提交的内容比较

```
git diff commintId
```

## 查看提交

查看历史提交

```
git log
```
查看某个文件的提交记录

```
git log -- filename
```
查看某个文件每次提交的diff

```
git log -p filename
```

查看简略的历史提交

```
git reflog
```
查看上次提交的内容

```
git show
```
查看历史提交的内容

```
git show commitId
```

## 分支

列出所有本地分支

```
git branch
```
列出所有远程分支

```
git branch -r
```
列出所有本地和远程分支

```
git branch -a
```
显示本地分支的提交状态

```
git branch -v
```
显示本地分支与远程分支的对应关系

```
git branch -vv
```
显示已合并到当前分支的分支

```
git branch --merged
```
显示未合并到当前分支的分支

```
git branch --no--merged
```

创建分支
```
git branch 分支名
```
切换分支
```
git checkout 分支名
```
基于当前分支创建并切换分支

```
git checkout -b 分支名
```
基于远程分支创建

```
git checkout -b 本地分支名 远程分支名
```
删除分支

```
git branch -d
```
强制删除分支

```
git branch -D
```

## 总结

以上都是工作和学习中亲自用过的，常用而不全面，想了解更多玩法，详见：

```
git 命令 -h
```
#### **下篇给大家带来Git的进阶用法，感谢阅读！**


### 欢迎关注个人微信公众号：Charming写字的地方

















