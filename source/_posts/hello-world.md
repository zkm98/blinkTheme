---
title: Hello World to Hexo
date: 2020-09-11 20:26:00
author: zhangyuanes
categories: 博客搭建
tags:
  - Hexo
  - Gitpage
---

## 说在前面

我的blog很大程度上不算blog，算是个整合贴。我的blog的流程一般是这样的：

遇到一个实际问题，或者从完全不了解到初步入门，我会保留自己在解决这个问题的浏览器历史搜索记录（请务必科学上网）、大致流程、特殊问题和解决方案。

一般不再书写别人已经写作的内容，仅仅贴上链接。虽然不排除链接实效的情况，但是一般来说链接是稳定的。万一实在链接失效，也可以搜索关键词找寻更新的教程。在此不再赘述。

## [使用Github Pages和Hexo构建个人博客][gitpage_hexo]

此项目是基于gitpage的托管，在本地nodejs平台渲染hexo项目生成的静态网站。

静态博客文档的书写一方面是记录了博主的工作，另一方面在线分享也节省了有同样问题或者想要了解、从事某方面研究的人的调研时间，一举两得。

感谢所有愿意分享的博主，你们的无私让整个社区更加美好。

### [Valine无后端评论系统][Valine]

这是静态网站中不那么静态的部分——评论系统。

后台存储使用的是LeanCloud。详细配置文档中有细致说明。

## [使用jekyll构建gitpage静态网页][gitpage_jekyll]

这是另一种实现静态网站的方案，但是jekyll是基于ruby的，对于windows用户可能不是那么友好，所以推荐使用前面的方案。

## [Ubuntu安装Node环境][installNode]

windows下直接安装就OK。这里采用了NVM的Node版本管理来做安装,并使用国内镜像。

### [NVM github][NVM github]

## 仓库主题下载使用

[仓库地址][repo],请参照 readme运行，参照 themes\hexo-theme-matery\README_CN.md 中说明配置详细主题信息.


[gitpage_hexo]:https://developer.aliyun.com/article/387750
[Valine]:https://valine.js.org/
[gitpage_jekyll]:https://sspai.com/post/54608
[installNode]:https://mupceet.com/2020/02/the-best-way-to-install-nodejs/
[NVM github]:https://github.com/nvm-sh/nvm#installing-and-updating
[repo]:https://github.com/zkm98/blinkTheme