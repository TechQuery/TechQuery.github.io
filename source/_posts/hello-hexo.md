---
title: Hello Hexo!
author: 南漂一卒
categories:
  - Technology
tags: 
  - Hexo
  - Travis
---

[GitHub `static-site-generator` 主题下 Node.JS 架构](https://github.com/topics/static-site-generator?l=javascript&o=desc&s=stars) star 前三的项目中，[Hexo](https://hexo.io/) 生态最完善：

 1. 包括**中文**在内的**多语言官网**

 2. 使用文档不但每页开篇有**视频讲解**，简体中文视频直接托管在 Bilibili

 3. 插件、主题的**搜索引擎**更是方便，接地气的**国人作品**也很多

官网一番学习之后，发现起步比较简单：

```shell
npm install hexo --global

hexo init my_pages

git init
git checkout --orphan hexo
```
随后又在官网搜了一个简洁、美观、强大、易用的主题模板 [Matery](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)。

最后一顿 Google，入门了近几年开源界最流行的 [Travis CI](https://travis-ci.com/)：

```yaml
branches:
  only: 
    - hexo

env:
  global:
    - GIT_URI: github.com/yourID/yourID.github.io

language: node_js
node_js: 
  - lts/*
cache:
  directories:
    - node_modules

install:
  - npm install
script:
  - npm run build
after_script:
  - cd public
  - git init
  - git config user.name "TechQuery"
  - git config user.email "shiy2008@gmail.com"
  - git add .
  - git commit -m ":memo:\ Update HTML by Travis CI"
  - git push --force --quiet https://${TOKEN}@${GIT_URI}.git master:master
```
新版个人网站初步落成！~

---

 1. https://segmentfault.com/a/1190000013058880

 2. https://ssk7833.github.io/blog/2016/01/21/using-TravisCI-to-deploy-on-GitHub-pages/