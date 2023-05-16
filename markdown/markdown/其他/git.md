# 配置文件

```shell
$ git config --global user.name "username"
$ git config --global user.email "email@example.com"
```
# 初始化仓库

```shell
$ git init
Initialized empty Git repository in /Users/hxk/mymenu/.git/
```
#  添加文件到暂存区

```shell
$ git add test.txt
```
# 提交文件到仓库

```shell
$ git commit -m "a new file"

#-m后面输入的是本次提交的说明

#为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件

#如果提交的备注写错了，可以用以下命令修改刚刚提交的备注
$ git commit --amend
```
# 查看历史记录

```shell
$ git log

#git log显示最近到最远的提交日志，我们可以看到两次提交，最后一次是append ABC

#git的版本号是用SHA1计算出来的一个16进制数

#如果嫌输出信息太多，可以加上--pretty=oneline
```
# 回退历史版本

```shell
$ git reset --hard HEAD^
HEAD is now at eaadf4e a new file
```
首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，上一个版本就是HEAD\^，上上一个版本就是HEAD\^\^，当然往上100个版本写100个\^比较容易数不过来，所以写成HEAD~100

回退上一版本：

```shell
$ git reset --hard HEAD^
```
# 查看历史命令

```shell
$ git reflog
```
# 查看状态

```shell
$ git status
```
# 使用gitee远程同步
## git全局设置
```shell
git config --global user.name "username"
git config --global user.email "email"
```
## 本地创建git仓库
```shell
cd test
git init 
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com
git push -u origin "master"
```
## 已有仓库?
```shell
cd existing_git_repo
git remote add origin git@gitee.com
git push -u origin "master"
```
# 通过ssh连接gitee
生成公钥
```shell
ssh-kengen -C "邮箱"

#按照提示完成三次回车，即可生成 ssh key。通过查看 ~/.ssh/id_ed25519.pub 文件内容，获取到你的 public key# 
```
复制生成后的 ssh key，通过仓库主页 「管理」->「部署公钥管理」->「添加部署公钥」 ，添加生成的 public key 添加到仓库中

添加后，在终端（Terminal）中输入

```shell
ssh -T git@gitee.com
```
首次使用需要确认并添加主机到本机SSH可信列表。若返回 Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access. 内容，则证明添加成功

之后，在git push操作时可以使用git仓库的ssh地址，无需输入用户名和密码：

```shell
git push git@gitee.com:wohtnaw/markdown.git
```
# 使用shell脚本和crontab定时任务实现定时自动提交
## 编辑gitpush.sh脚本

```shell
#!/bin/bash
cd /home/wanthw/"Nutstore Files"/Markdown
git add *
git commit -am "自动提交" #-a 参数会在提交到本地仓库的时候，会先更新修改和删除的文件信息到暂存区（它不会提交新增加的文件信息）
git push git@gitee.com:wohtnaw/markdown.git


#编辑完成后，将以上脚本移动到指定路径下
mv gitpush.sh 脚本路径
```
## 将以上脚本文件定时执行

```shell
#创建定时任务
crontab -e

#输入以下内容并保存
* * * * * /bin/bash 脚本路径
```
