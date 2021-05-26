# blinkfox Theme fix to myself

refers by https://github.com/zhangyuanes/blinkTheme/blob/master/README.md

[![Build Status](https://secure.travis-ci.org/blinkfox/blinkfox.github.io.svg)](https://travis-ci.org/blinkfox/blinkfox.github.io) [![GitHub license](https://img.shields.io/github/license/blinkfox/blinkfox.github.io.svg)](https://github.com/blinkfox/blinkfox.github.io/blob/hexo/LICENSE) [![GitHub forks](https://img.shields.io/github/forks/blinkfox/blinkfox.github.io.svg)](https://github.com/blinkfox/blinkfox.github.io/network) [![GitHub stars](https://img.shields.io/github/stars/blinkfox/blinkfox.github.io.svg)](https://github.com/blinkfox/blinkfox.github.io/stargazers)

根据自己需求进行自定义修改和美化的blinkfox的hexo主题。

> 我的 [personal blog](https://zhangyuanes.github.io/) ，可以点击查看预览。

## How to Use

安装NodeJS（Windows，Linux，MacOS均可）。

### Fork仓库后使用git克隆项目

点击fork按钮fork到自己的账号下，这样你的配置就可以保存下来了。

```
git clone https://github.com/zhangyuanes/blinkTheme.git
```

###  进入项目文件夹，安装依赖包

```
cd blinkTheme
npm install
```

###  清理并生成渲染

```
npm run clean
npm run build
```

###  本地预览

```bash
npm run server
```

访问`http://localhost:4000`即可看到页面。

###  使用hexo-admin做后台dashboard

预先配置有hexo-admin，在url结尾加入`/admin`即可进入管理页面，可以在线md交互预览编辑。用户名和密码我配置的为：yanbo，你可以修改配置文件来自定义，相关配置在`_config.yml`中:

```yml
# hexo-admin authentification
# 在本地浏览的时候在url之后加上admin即可访问
# 这里的password是密码经过sha256加密
admin:
  username: yanbo
  password_hash: $2a$10$SD3chEWmZ4/qWCOOvmVv3ut5/lKgPqDx5YBLwtZHt07/XzcG4TEAK
  secret: yanbo
  deployCommand: 'hexo-deploy.bat'
```

###  推送到github.io

如果你没有设置这个gitpage仓库，[参考这里](https://zhangyuanes.github.io/2020/09/11/hello-world/).

需要预先设置项目根目录下的`_config.yml`中的部署deploy配置：

```
# Deployment
deploy:
  type: git
  repo: https://github.com/********/********.github.io.git
  branch: master
```

预览渲染没有问题可以使用命令推送到gitpage，推从到github.io仓库中，使用命令：

```bash
npm run deploy
```

当显示成功后刷新仓库，就可以访问到对应的页面了，页面地址为 `https://YourGithubName.github.io`。

### 编写你的文章

清除`source/_posts`下全部文章页（但请至少保留一个md文件用于生成页面，否则build会失败），完成`_config.yml`中其他个性化配置后重新清理并生成渲染，预览后推送即可。

`_config.yml`已经添加很多中文注释，如果需要请按照注释修改即可。

本项目代码唯一需要用户单独存档的仅仅为`source/_posts`下的原始md文章页面以及对应的配图。

配图建议使用图床，这样就不用担心相对引用，相关文章[参考这里](https://zhangyuanes.github.io/2021/01/19/ji-lu/bo-ke-da-jian/tu-chuang-da-jian/)。本仓库的配图还是比较大的，后续会逐渐修改为图床链接。

补：如果不是图床的图片，配图请放在`source/medias`中，如果需要分类请在此文件夹下新建文件夹放置即可，在页面中引用地址为：`/medias/******.jpg`。

md文章的编写如果不清楚可以先参考我的md，里面基本内容包括：header和正文。

header需要用三个连接号显式表示出来，字段和含义如下 ： `title` 文章标题, `date` 时间, `author` 作者, `categories` 分类,`tags` 文章标签。

header写完后就是正文，直接兼容全部的md语法，自由书写即可。

```md
---
title: Hello World to Hexo
date: 2020-09-11 20:26:00
author: zhangyuanes
categories: 博客搭建
tags:
  - Hexo
  - Gitpage
---

<你的文章内容>

```

### 如有其他问题请提交issue或发邮件询问。
## License

[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)
