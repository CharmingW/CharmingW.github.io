## 前言

这次给大家带来Git进阶用法第二篇，上一篇基础篇还没看的同学可以去看看，传送门：[Git教程——高级进阶（一）]()


## 远程

以下默认远程仓库名为origin，远程仓库地址为url，主分支为master

#### clone
clone远程仓库到本地
```
git clone url
```
#### remote
添加远程仓库
```
git remote add origin url
```
删除远程仓库

```
git remote rm origin
```
查看远程仓库地址

```
git remote -v
```

修改远程仓库地址，有三个选项：--add、--delete、--push 

```
git remote set-url
```

#### push
推送更改到远程仓库

```
git push
```
若本地分支和远程分支没有对应上，则可以

```
git push --set-upstream origin master
```
或

```
git push -u origin master
```
将本地分支和远程分支关联

#### pull
拉取远程更改到本地并合并

```
git pull
```
#### fetch
拉取远程更改到本地但部合并
```
git fetch 
```

#### ssh
使用http或https与远程仓库通信每次都需要输入密码，稍微有点麻烦，那么可以用ssh方式通信，以github为例  

先在本机生成密钥对

```
ssh-keygen -t rsa -C "xxx@gmail.com"
```
之后会在`/用户文件夹/.ssh/`下生成id_rsa和id_rsa.pub两个文件  

到Github的Settings，选择SSH and GPG keys，点击New SSH key，命名，复制id_rsa.pub内容到密钥输入框，完成创建，之后本机和远程通信就无须密码了


## 总结

本篇主要讲解git远程仓库的一些常见操作，更多有趣的操作，请参考git帮助文档





