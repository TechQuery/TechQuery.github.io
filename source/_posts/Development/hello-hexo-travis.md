---
title: Hello Hexo & Travis !
date: 2018-12-29 19:30:00
updated: 2019-11-22 02:16:52
author: 南漂一卒
categories:
  - Development
tags:
  - Hexo
  - Travis
  - Azure
  - DevOps
---

## 静态网站生成器

[GitHub `static-site-generator` 主题下 Node.JS 架构][1] star 前三的项目中，[Hexo][2] 生态最完善：

1.  包括**中文**在内的**多语言官网**

2.  使用文档不但每页开篇有**视频讲解**，简体中文视频直接托管在 Bilibili

3.  插件、主题的**搜索引擎**更是方便，接地气的**国人作品**也很多

官网一番学习之后，发现起步比较简单：

```shell
npm install hexo --global

hexo init my_pages

git init
git checkout --orphan hexo
```

随后又在官网搜了一个简洁、美观、强大、易用的主题模板 [Matery][3]。

## 持续集成、持续部署

最后一顿 Google，入门了近几年开源界最流行的 [Travis CI][4]，它的配置文件 `.travis.yml` 看起来像 `Dockerfile`，但更自然：

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

before_install:
  - export TZ=${TIME_ZONE}
install:
  - npm install
script:
  - npm run ${SCRIPT}
deploy:
  provider: pages
  on:
    branch: hexo
  skip_cleanup: true
  local_dir: ${FOLDER}
  fqdn: ${DOMAIN}
  token: ${TOKEN}
  target_branch: ${BRANCH}
```

上述命令行脚本中的**环境变量**可在 Travis CI 项目配置页设置，示例如下：

|  变量名   |     示例值     |         释义          |                  备注                  |
| :-------: | :------------: | :-------------------: | :------------------------------------: |
| TIME_ZONE | Asia/Chongqing |       系统时区        |                                        |
|  SCRIPT   |     build      |      构建脚本名       |                                        |
|  FOLDER   |     public     | 网页生成器的输出目录  |                                        |
|  DOMAIN   |  example.com   |      自定义域名       |                                        |
|   TOKEN   |                | Git 平台个人访问令牌  | https://github.com/settings/tokens/new |
|  BRANCH   |     master     | GitHub Pages 目标分支 |                                        |

新版个人网站初步落成！~

## 开源项目 API 文档生成

既然 Travis CI 配置脚本可以用环境变量变得灵活，那基于 JSDoc、ESDoc 之类的开源项目生成 **API 文档站**也可照搬以上方法了：

- 再也不用在本地 Git `precommit` 钩子上执行**文档生成脚本**了，加快提交速度

- 把生成的文档放在独立的 `gh-pages` 分支，让 `master` 分支只放源码，**提交记录**更清爽

- 用 `git push --force` 也让**文档站分支**不保留不必要的提交记录，**仓库体积**最小化

但是，Travis 的 Windows 环境尚处测试阶段，我实测时出现“失败但无报错详情”的 bug，只好让 [Puppeteer-IE][5] 改用 [Azure Pipeline][6]：

```yaml
trigger:
  - master

pool:
  vmImage: 'vs2017-win2016'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '8.x'
    displayName: 'Install Node.js'

  - bash: |
      npm install
      npm run build
      cd ${DOC_FOLDER}
      git init
      git config user.name ${GIT_USER}
      git config user.email ${GIT_EMAIL}
      git add .
      git commit -m "${GIT_MESSAGE}"
      git push --force --quiet https://${GIT_TOKEN}@${GIT_URI}.git master:${GIT_BRANCH}
    displayName: 'npm install & build Document'
```

再次领略 微软文档的一大特点 —— 要么一笔带过、不知所云，要么又臭又长、不明重点…… 好歹有个**项目配置模板**，连蒙带猜改一改，竟然能用…… 也算是不枉费我[秋天跑一趟上海][7]~

## 参考资料

1.  https://segmentfault.com/a/1190000013058880

2.  https://ssk7833.github.io/blog/2016/01/21/using-TravisCI-to-deploy-on-GitHub-pages/

3.  https://easyhexo.github.io/Easy-Hexo/

4.  https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=vsts&tabs=example#bash

[1]: https://github.com/topics/static-site-generator?l=javascript&o=desc&s=stars
[2]: https://hexo.io/
[3]: https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md
[4]: https://travis-ci.com/
[5]: https://tech-query.me/Puppeteer-IE/
[6]: https://azure.microsoft.com/zh-cn/services/devops/pipelines/
[7]: https://www.microsoft.com/china/techsummit/2018/
