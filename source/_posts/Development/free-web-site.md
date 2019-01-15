---
title: 云计算时代的免费建站
date: 2019-01-09 01:40:00
author: 南漂一卒
categories:
  - Development
tags:
  - Web
  - free
  - cloud
  - BaaS
  - GitHub
---


**云计算**看起来更高大上，所以在这个时代**网站建设**更难了吗？

不！越分工应该越简单！

回望“免费空间”的时代，建站不过是“FTP 上传”，而今 HTTPS 证书、SSH 密钥、NGINX 配置、域名备案等等又繁琐又费钱的步骤，真让人掉发！

有没有办法更简单？有，但需要一番主动的探索……


## 静态网页

静态网页托管认准 [GitHub Pages](https://pages.github.com/)，需要 **MarkDown 转 HTML** 的推荐 [Hexo](/technology/programming/hello-hexo-travis/)。


## 顶级域名

[Freenom](https://www.freenom.com/) 托管的顶级域名后缀简短、申请方便，Google 账号一键登录后即可搜索可用域名。一次最多**免费租用 12 个月**，到期前有电邮提醒续期，可**免费续租 12 个月**。若忘了续租，系统就认为是有价值的域名，就只能美刀去买了。


## HTTPS 证书

[CloudFlare](https://www.cloudflare.com/?r=1) 是全球著名的域名、CDN 服务提供商，**Web 前端界**国外使用最多、国内借鉴最多的 [CDNJS](https://cdnjs.com/) 便是它赞助的。用了它，**免费 HTTPS 证书**、**DNS 管理**、**静态资源 CDN** 一次性搞定！以后还可以用它的收费 **DDoS 攻击防御**，久经骇客攻击考验~

注册账号并登录后，即可一键导入**域名基本配置**，再去 Freenom 后台把 Name server 重定向过来，就可以用 CloudFlare 更先进的域名管理后台了。至于 HTTPS 配置，直接参考[官方博文](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/#step2settingupourdns)，配好后要等一会儿才能让国内 DNS 查到。


## CDN 加速

既然用了 CF，为何还要自己做 CDN？因为 CF 在中国大陆与*百毒熊厂*合作业务，我是极端恶心*李厂长夫妇*的尿性的，只用 **CloudFlare 海外版**，有受众在天朝就得另辟蹊径。

我最终选用国内 **BaaS 云服务**龙头 [LeanCloud](https://leancloud.cn/)：

 1. LeanCloud 云服务在天朝 **Ping 延时** < 100ms，并能在**应用管理后台**看到流量响应统计

 2. **应用二级域名**是免费、无需备案的，在每个 HTML 文件 `<head />` 标签最前面加个 `<base href="https://my-app.leanapp.cn/">`，**用顶级域名打开页面、用二级域名加载外置资源**二者各司其职

 3. 每个**云应用**都能启动一个**免费容器实例**，虽然每天强制休眠 6 小时，但对绝大多数小网站够用，实在不够也可开个**￥1/日的生产实例**

 4. 底层基于 [Docker](https://www.docker.com/) 支持标准的 Node.JS、Python、Java、PHP、C# 项目，**纯 Web 前端项目**只需用 NPM 装个[命令行版 Web 服务器包](https://tech-query.me/KoApache/)即可架站

 5. **应用代码**还能直接用 [Web hook](https://developer.github.com/webhooks/) 从一个 **Git URL** 拉取并自动部署

 6. 官方封装的**数据存储**、**手机短信**、**即时通讯**、**消息推送**、**游戏客户端引擎**等云服务也有方便的 API、SDK，为今后更多需求的开发提供保障


---

【整套方案的样板项目】https://github.com/FreeCodeCamp-Chengdu/WFE-Conf
