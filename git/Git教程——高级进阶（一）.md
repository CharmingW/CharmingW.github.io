## 前言

这次给大家带来Git稍微高级一点的用法，上一篇基础篇还没看的同学可以去看看，传送门：[Git教程——入门基础]()

## 撤销

#### 撤销工作区的修改  

单个文件
```
git checkout HEAD README.md
```
多个文件
```
git checkout HEAD README.md SUMMARY.md
```
所有文件
```
git checkout HEAD .
```

关于 `checkout` ，除了切换分支、撤销修改以外，还有一个很好的用法，就是快速查看项目旧版本，比如

```
git checkout commitId
```
通过传入历史提交的 id，将 HEAD 移动到相应的历史提交处，方便查看历史代码

#### 取消暂存

单个文件
```
git reset HEAD README.md
```
多个文件

```
git reset HEAD README.md SUMMARY.md
```
所有文件

```
git reset HEAD .
```
取消暂存并撤销工作区的修改，以所有文件为例

```
git reset --hard HEAD .
```

复位到上上次提交（HEAD代表上次提交）

```
git reset --hard HEAD^^
```
复位到历史某一次提交

```
git reset --hard commitId
```

#### 反转提交

有时候你可能想撤销某次提交，但不想改变提交历史，这时候可以用`revert`  
Revert：撤销一个提交的同时会创建一个新的提交。这是一个安全的方法，因为它不会重写提交历史。

`git log` 查看历史提交

```
commit e19c68095958937387cfb7ccd9a870448e7d4495 (HEAD -> master)
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:54 2017 +0800

    third

commit 0f90338c7bbeced9e486e5c4448881316538b3e8
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:37 2017 +0800

    second

commit 2695da1b0503c514a476f126ecc8b8ec2847de1c
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 14:52:59 2017 +0800

    first

```

你想对撤销 `third` 这次提交，你可以

```
git revert HEAD
```
再 `git log` 查看提交

```
commit 9073bf136a0f4be829834ce85b9b293007d27067 (HEAD -> master)
Author: Charming <cm19951018@gmail.com>
Date:   Mon Nov 13 15:09:30 2017 +0800

    Revert "third"

    This reverts commit e19c68095958937387cfb7ccd9a870448e7d4495.

commit e19c68095958937387cfb7ccd9a870448e7d4495
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:54 2017 +0800

    third

commit 0f90338c7bbeced9e486e5c4448881316538b3e8
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:37 2017 +0800

    second

commit 2695da1b0503c514a476f126ecc8b8ec2847de1c
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 14:52:59 2017 +0800

    first

```
发现多了一次 `Revert "third"`的提交，这次提交为了撤销 `third` 而创建的，属于一次反提交。这样，既达到了撤销的效果，也保存了历史提交，这种用法一般在主分支上


## 变基

当你想对历史某一次提交进行修改，又不想增加新的提交时，可以使用变基操作。  
比如：你正在进行开发，你发现有个参数名写错了，属于小问题，而这个参数正是你历史提交中某一次引入的，你想修改这个参数名而不产生新提交，这时候可以使用变基

```
git rebase -i commitId
```
先查看历史提交 `git log`

```
commit 87ca1e73c3cf59966f07c6ee657af4ade18a8061 (HEAD -> master)
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:54 2017 +0800

    third

commit b1fb0abe4355f2df1d607b65b326d3b79a32772d
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:37 2017 +0800

    second

commit 2695da1b0503c514a476f126ecc8b8ec2847de1c
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 14:52:59 2017 +0800

    first
```

你想对 `second` 这次提交进行修改，你可以变基到 `first` 提交，也就是以 `first`提交为基础（想修改最近n次提交的其中一次，可以使用 `git rebase -i 最近第n+1次提交的id`列出最近n次提交，然后挑选要修改的提交）  

因此

```
git rebase -i 2695da1b0503c514a476f126ecc8b8ec2847de1c
```
这时候会出现

```
pick 0f90338 second
pick e19c680 third

```
修改为

```
edit 0f90338 second
pick e19c680 third
```
保存退出，`git log`

```
commit 0f90338c7bbeced9e486e5c4448881316538b3e8 (HEAD)
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 15:17:37 2017 +0800

    second

commit 2695da1b0503c514a476f126ecc8b8ec2847de1c
Author: Charming <cm19951018@gmail.com>
Date:   Sun Nov 12 14:52:59 2017 +0800

    first
    
```
这时候你可以对 `second` 提交进行修改，然后增补提交

```
git add .
git commit --amend
```
修改完提交，结束变基，回到最新的提交结点

```
git rebase --continue
```
若是想取消变基操作，放弃修改，则可以

```
git rebase --abort

```

## 总结

`checkout`、`reset`、`revert` 和 `rebase` 是几个非常容易混淆的命令，通过学习理解它们对工作区、暂存区和提交历史的影响，想必现在你已经对在什么场景使用什么命令已经非常清楚了吧  

下面这个表格总结了这些命令最常用的使用场景。记得经常对照这个表格，因为你使用 Git 时一定会经常用到

命令	|作用域|	常用情景
-|-|-
git reset|	提交层面|	在私有分支上舍弃一些没有提交的更改
git reset|	文件层面|	将文件从缓存区中移除
git checkout|	提交层面|	切换分支或查看旧版本
git checkout|	文件层面|	舍弃工作目录中的更改
git revert|	提交层面|	在公共分支上回滚更改
git revert|	文件层面|	无















