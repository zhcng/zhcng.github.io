---
title: 一步一步使用Hexo搭建Github博客
date: 2017-04-25 10:21:52
categories: Hexo
tags: Yelee
---
最近看到不少人在使用Hexo搭建自己的个人博客，很感兴趣。于是找了一些资料，踩了一些坑，也搭建了属于自己的博客。以下为搭建的步骤(Windows环境)。

<!-- more -->

## 前期环境准备

### 安装Git

直接到[Git官方下载地址](https://git-for-windows.github.io/)下载并进行傻瓜式安装即可。

### 安装Node.js

同样，直接到[Node.js官方下载地址](https://nodejs.org/en/)下载并进行傻瓜式安装即可。

### 安装Hexo

任意位置使用Git bash，利用以下npm命令即可完成安装。

``` bash
npm install -g hexo
```

## 本地搭建

### 创建文件夹

选择你需要安装的文件夹，比如D:\hexo，使用Git bash 执行以下指令，便会自动在该文件夹中导入建立网站所需文件。

``` bash
hexo init
```

### 安装依赖包

接着，还是在D:\hexo的Git bash 中执行以下命令，安装依赖包。

``` bash
hexo install
```

### 运行本地服务器

最后，在E:\hexo的Git bash中执行以下命令，生成网页，启动服务器。

``` bash
hexo generate
hexo server
```

现在，本地的网站就已经搭建好了，在浏览器中输入[localhost:4000](http://localhost:4000)即可查看我们的网站了。

## 配置Github

### 注册Github账号

到[Github官网](https://github.com/)，填写用户名、邮箱、密码即可完成注册。

### 创建仓库

注册完后就可以点击New repository，新建一个仓库。这里要注意，仓库的名称，必须是你的用户名.github.io，比如我的仓库名为：zhcng.github.io。

### 添加SSH密钥

首先得明确一点，这一步不是必须的。如果不添加，每次博客有改动提交的时候就需要手动输入账号密码，配置了就不需要了。所以建议可以在常用的个人电脑上添加一下。

SSH公钥默认在C:\Users\Administrator``\``.ssh目录下，如果没有这个目录，需要手动生成一下。直接到Git的程序目录里，运行git-bash.exe，然后运行以下指令。

``` bash
cd ~
```

这条指令中的邮箱名填写自己的邮箱，打完指令根据提示连续三个回车，设置密码为空即可。

``` bash
ssh-keygen -t rsa -C "artistzheng@163.com"
```

注意，ssh与-keygen之间是没有空格的，否则会报 Bad escape character 'ygen'。

运行完指令后，在C:\Users\Administrator``\``.ssh目录下找到id_rsa.pub，复制里面所有字符，到[Github设置SSH](https://github.com/settings/ssh )的页面,New SSH key，粘贴进去就添加完毕了。

## 最终部署

### 全局配置

首先，先来了解一下hexo的主要目录结构。

- .deploy_git ``#``需要部署的文件
- node_modules ``#``hexo插件
- public ``#``生成的静态网页
- scaffolds ``#``模板
- source ``#``源文件（博客正文、404页面等）
- themes ``#``主题
- _config.yml ``#``全局配置文件

这里我们要配置的就是_config.yml文件。需要注意的常用修改如下。

```
title: 博客--Chen #标题
subtitle: 诗酒趁年华 #副标题
description: Chen的个人博客 #为搜索引擎提供的站点描述
author: Chen #作者
language: zh-Hans #语言（中文简体）
url: http://zhcng.github.io #自己的链接
theme: landscape #主题
deploy:
  type: git
  repo: https://github.com/zhcng/zhcng.github.io.git #自己新建的git仓库地址
  branch: master
```

### 添加文章

首先，运行以下指令新建一篇文章，会在source/_posts目录下生成标题.md。

``` bash
hexo new "标题"
```

打开文件用Markdown语法编辑正文。完成保存后运行以下指令进行本地预览。

``` bash
hexo server
```

这里再次运行部署前，建议先清理缓存，所以上面的命名行可以用下面的代替。

``` bash
hexo clean && hexo s
```

最后，运行以下指令生成网页，完成部署。

``` bash
hexo generate
hexo deploy
```

如果deploy遇到Deployer not found: git问题，运行以下指令，然后重新deploy。

``` bash
npm install hexo-deployer-git --save
```

出现[info] Deploy done: git，表明部署成功。此时打开用户名.github.io就可以看到自己的博客了。

## 更换主题

我们可以为hexo更换各种已有的主题。这里我使用的是Yelee主题，下面就这个主题的更换做简单的说明，更详细的教程可以[查看这里](http://moxfive.coding.me/yelee/)。

### 安装主题

在E:\hexo\themes文件夹下运行以下指令，下载主题。

``` bash
git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee
```

再到E:\hexo下找到配置文件_config.yml，修改theme属性为Yelee。此时重新运行服务器即可预览当前主题。

### 设置头像

默认头像存储路径在D:\hexo\themes\yelee\source\img，配置是在D:\hexo\themes\yelee中的_config.yml中的avatar属性。注意这里是主题的配置文件，而不是hexo的配置文件。

### 设置站点小图标

可以跟头像一样，将站点小图标放在D:\hexo\themes\yelee\source\img。在主题配置文件中修改favicon属性进行对应即可。站点小图标可以在[这里](http://www.bitbug.net/)进行制作。

### 设置主菜单

在主题的配置文件中找到menu属性，下面列出来的就是主菜单的条目，可以通过在开头加``#``隐藏掉你不想要的条目。

### 添加关于我页面

使用以下命令，新建一个名为about的页面。

``` bash
hexo new page about
```

然后在D:\hexo\source\about\index.md文件夹中修改页面内容即可。

### 显示摘要

有两种方法可以使文章在首页显示摘要而不是全文。

一种是在正文中增加``<!-- more -->``标签，这样标签前的文字就能显示在首页中。

还有一种是在Front-matter中添加description属性，这样就可以在首页中显示属性的内容。


## 总结

好了，到了这里，应该一个有很不错的风格的博客已经搭建起来了。你可以通过用户名.github.io来访问这个网站。并且，只需要你有简单的[MarkDown基础](http://www.jianshu.com/p/q81RER/)，就可以很方便的进行博文的添加与编辑了。


