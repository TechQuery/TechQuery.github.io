---
title: 【原创】Docker LAMP 入门
date: 2015-11-17 03:54:00
author: 南漂一卒
categories:
  - Development
tags:
  - Docker
  - LAMP
  - server
  - cloud
---


**LAMP 服务器套件**（Linux + Apache + MySQL + PHP）早已成为 **Web 应用开发者** 最常用的**部署环境**，几乎是入门必修课，但它的搭建、维护也是对初学者很艰难的一课。虽然有 **XAMPP** 这样的跨平台一键安装包，但它们主要是为了方便搭建开发环境，并不适合直接用在**线上生产环境**。

自己辛辛苦苦写好的网站上线都费事，简直太沮丧了…… 于是，各种各样的**云计算服务**应运而生，从各个层面解决不同水平 Web 应用开发者的部署需求。下面是从服务底层到用户顶层依次列出的常见**云计算服务模式** ——

 - **IaaS**（**基础设施** 即 服务）—— 云主机/服务器
 - **CaaS**（**应用容器** 即 服务）—— Docker
 - **PaaS**（**技术平台** 即 服务）—— AWS、GAE、SAE 等
 - **BaaS**（**后端 API** 即 服务）—— LeanCloud 等
 - **SaaS**（**应用软件** 即 服务）—— Tower、Worktile 等

在云计算时代到来之前，还有几种常见的部署环境 ——

 - 类似 PaaS 的**网站空间** —— 本质上是一个操作系统中用 Apache 之类的 HTTP 服务器，对不同的域名 指向不同的**网站根目录**，所以上传网站文件只能用 FTP 读写你自己的目录，网站程序也不能读写本站之外的服务器目录，同机的所有网站共享一个 IP 地址
 - 类似 IaaS 的 **VPS（虚拟主机）** —— 同样是独享一个操作系统的虚拟机，但究其**宿主环境**，一个 VPS 只能运行在一个真实的服务器主机上，而云主机则运行在一群真机协作抽象成的一个超级服务器（**服务器集群**）。后者的硬件资源 超高利用率、真机热插拔（便于扩展、容错）等特性，是前者望尘莫及的；但对它们的用户来说，包括 **独享 IP**、**SSH**（远程登录命令行）等关键特性在内的使用方法几乎完全一样

至于 **Docker**，它旨在**把 Web 应用程序和它的标准运行环境（由一个配置文件描述）打包为一个虚拟机镜像**，让部署、迁移、扩展一键搞定，无需开发人员、运维人员手动配置云主机，因“开发与部署的环境不完全相同”而产生的 Bug 也没有了。

---

构建一个 **Ubuntu + LAMP** 镜像的 **Dockerfile** 配置文件 ——

```dockerfile
FROM ubuntu:14.04

MAINTAINER test_32 "shiy2008@gmail.com"


RUN apt-get update; \
    apt-get install -y lamp-server^;

COPY . /var/www
EXPOSE 80

CMD /etc/init.d/apache2 restart
```

---

【参考文档】

 1. http://blog.csdn.net/we_shell/article/details/38445979
 2. http://blog.sina.com.cn/s/blog_7380598c0100wdl5.html
 3. http://segmentfault.com/q/1010000000693754
 4. http://dockerfile.github.io/#/ubuntu
 5. http://www.alauda.cn/2015/07/12/dockerfiledemo/
 6. http://dockerpool.com/static/books/docker_practice/
 7. https://docs.docker.com/
 8. http://www.aixq.com/post-328.html
 9. http://blog.sina.com.cn/s/blog_4dc988240102vj8a.html
 10. http://www.docker.org.cn/book/docker.html
 11. http://my.oschina.net/ylchou/blog/522381
 12. https://github.com/docker/kitematic/issues/1037
 13. http://www.oschina.net/question/1171658_2177501
