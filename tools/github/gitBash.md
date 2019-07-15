# 1 利用git bash（git命令行）将本地代码上传到github上

[Windows10下安装Git](https://blog.csdn.net/qq_32786873/article/details/80570783)

Git是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。具体安装步骤如下：

第一步：先从官网下载最新版本的Git

官网地址：https://git-scm.com/downloads



-----------------------------------------------------------------------------------------------------------

git命令工具Git Bash 。 

首先在Git Bash中使用cd命令进入对应的本地项目路录，按照下面的命令操作： 
1、git init 表示在当前的项目目录中生成本地的git管理。

2、git add . 表示你要提交到github上的文件，如果你要将所有文件都添加上去的话，使用git add . “.”表示添加当前目录中的所有文件。

3、git commit -m “first commit”,表示你对这次提交的注释。

git commit -a -m “提交的描述信息” 
git commit 命令的-a 选项可只将所有被修改或者已删除的且已经被git管理的文档提交倒仓库中。如果只是修改或者删除了已被Git 管理的文档，是没必要使用git add 命令的。

4、git remote add origin https://github.com/huangtianyu/mytest 就是项目地址。需要事先在github上创建好项目库。

5、git push -u origin master 用于将本地分支的更新，推送到远程主机，最后根据提示输入用户名和密码。

