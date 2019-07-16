# 第2节 GitBook使用

## GitBook在Windows上安装及使用

GitBook是基于Nodejs，使用Git/Github和Markdown制作电子书的命令行工具。

1、安装Nodejs

　　首先，安装Nodejs，官网地址：https://nodejs.org/en/

　　安装完成后输入命令node -v检测是否安装成功

2、安装全局Gitbook

　　在Nodejs安装目录下打开命令控制台，输入npm install gitbook-cli -g

　　由于安装默认采用国外镜像，所以需要等待一段时间。也可以使用国内镜像

　　打开nodejs安装文件夹下面的子目录E:\nodejs\node_modules\npm，找到里面的npmrc文件

​		加入以下配置信息：

　　　　registry=http://registry.npm.taobao.org

3、初始化

　　在任意文件夹下执行命令gitbook init，最终会生成README.md、SUMMARY.md两个文件。主要目录需要在SUMMARY.md文件中配置。

4、构建

　　执行命令gitbook build，会生成_book文件夹

5、启动

　　执行命令gitbook serve（也会执行构建工作），然后可以通过浏览器输入http://localhost:4000访问电子书目录



gitbook 在编译书籍的时候会读取书籍源码顶层目录中的`book.json`，存放配置信息。

其中可配置使用的插件，添加新插件之后需要运行`gitbook install`来安装新的插件。

Gitbook默认带有5个插件：

- highlight
- search
- sharing
- font-settings
- livereload