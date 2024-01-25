## git配置常用命令
### 1、代理配置
设置代理：
```bash
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```
取消代理：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 2、配置用户名和邮箱
配置用户名：
```bash
git config --global user.name "17383597845"
```
配置邮箱：
```bash
git config --global user.email "1743550834@qq.com"
```
查看配置：
```bash
git config user.name
git config user.email
```
### 3、查看所有配置
```bash
git config --list
```
### 4、github配置ssh key
https://blog.csdn.net/weixin_42310154/article/details/118340458
## 一、几个概念

### 1、工作区，暂存区，本地仓库，远程仓库，版本库
工作区：.git文件所在目录
暂存区：.git文件夹中有很多文件夹，其中idex文件就是暂存区（stage）。暂存区时一个临时保存修改文件的地方。
本地仓库：.git文件就是本地仓库，其中包含配置信息、日志信息和文件版本信息等。
远程仓库：存放到远程服务器上的仓库。
版本库：和本地仓库是一个概念。
### 2、文件状态
1、未被跟踪（untracked）
2、已被跟踪（tracked）
* 未修改（unmodified）：提交但提交后没有修改
* 已修改（modified）：提交后又修改了工作区内容
* 已暂存（staged）：添加到了暂存区
### 3、设置用户信息命令
```bash
#设置用户名
git config --global user.name '用户名'
#设置邮箱
git cofig --global user.email '邮箱'
#查看配置信息
git config --list
```


## 二、本地仓库操作
### 1、初始化仓库

```bash
#本地初始化
git init
#从远程仓库克隆
git clone 远程仓库地址
```
### 2、添加文件到暂存区

```bash
git add 文件名
```
### 3、提交暂存区内容到本地仓库

```bash
git commit
```
### 4、查看文件状态

```bash
git status
```
### 5、将暂存区文件取消暂存

```bash
git reset 文件名
```
### 6、切换工作区内容到指定版本

```bash
#版本号可以通过git log命令查看
git reset --hard 版本号
```
## 三、远程仓库操作
### 1、查看远程仓库

```bash
git remote
```
### 2、添加远程仓库

```bash
# shortname一般是origin可以通过git remote命令查看，url指的是远程仓库地址
git remote add <shortname> <url>
```
### 3、克隆远程仓库

```bash
#url：远程仓库地址
git clone <url>
```
### 4、将本地仓库内容推送到远程仓库

```bash
git push [remote-name] [branch-name]
```

## 四、分支操作
### 1、查看分支

```bash
# -r参数查看远程仓库分支，-a查看本地远程所有分支，不加参数查看本地分支
git branch <-r> <-a> 
```
### 2、创建分支

```bash
git branch [name]
```
### 3、切换到其他分支

```bash
git checkout [name]
```
### 4、推送分支到远程仓库

```bash
#shortname一般是origin name指要推送的分支名
git push [shortname] [name]
```
### 5、合并分支

```bash
#将name分支合并到此时所在的分支中
git merge [name]
```
## 五、标签操作
### 1、查看已有标签

```bash
git tag
```
### 2、创建标签

```bash
git tag [name]
```
### 3、将标签推送到远程

```bash
git push [shortname] [name]
```
### 4、检出标签

```bash
git checkout -b [branch] [name]
```
* 什么叫检出标签？
就是将标签状态克隆到新建的分支中
