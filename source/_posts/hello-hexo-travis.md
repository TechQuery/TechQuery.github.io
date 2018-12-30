---
title: Hello Hexo & Travis !
author: 南漂一卒
categories:
  - Technology
tags: 
  - Hexo
  - Travis
---


## 静态网站生成器

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


## 持续集成、持续部署

最后一顿 Google，入门了近几年开源界最流行的 [Travis CI](https://travis-ci.com/)：

```yaml
branches:
  only: 
    - hexo

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
  - cd ${FOLDER}
  - git init
  - git config user.name ${UID}
  - git config user.email ${EMAIL}
  - git add .
  - git commit -m "${MESSAGE}"
  - git push --force --quiet https://${TOKEN}@${GIT_URI}.git master:${BRANCH}
```
上述命令行脚本中的**环境变量**可在 Travis CI 项目配置页设置，示例如下：

| 变量名  | 示例值                              | 释义                  |
|:-------:|:----------------------------------:|:--------------------:|
| FOLDER  | public                             | 网页生成器的输出目录   |
| UID     | yourID                             | Git 用户名            |
| EMAIL   | yourID@email.com                   | Git 电邮地址          |
| TOKEN   |                                    | GitHub 个人访问令牌   |
| GIT_URI | github.com/yourID/yourID.github.io | Git 仓库标识符        |
| BRANCH  | master                             | GitHub Pages 目标分支 |
| MESSAGE | :memo: Update HTML by Travis CI    | Git 提交注记          |

新版个人网站初步落成！~


## 开源项目 API 文档生成

既然 Travis CI 配置脚本可以用环境变量变得灵活，那基于 JSDoc、ESDoc 之类的开源项目生成 **API 文档站**也可照搬以上方法了：

 -  再也不用在本地 Git `precommit` 钩子上执行**文档生成脚本**了，加快提交速度
  
 -  把生成的文档放在独立的 `gh-pages` 分支，让 `master` 分支只放源码，**提交记录**更清爽

 -  用 `git push --force` 也让**文档站分支**不保留不必要的提交记录，**仓库体积**最小化


## 参考资料

 1. https://segmentfault.com/a/1190000013058880

 2. https://ssk7833.github.io/blog/2016/01/21/using-TravisCI-to-deploy-on-GitHub-pages/