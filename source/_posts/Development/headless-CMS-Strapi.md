---
title: 内容型网站后端一把梭
date: 2020-04-09 20:05:18
updated: 2020-11-17 20:11:30
categories:
  - Development
tags:
  - CMS
  - API
  - Node.js
---

**Web 全栈工程师**经常遇到一些**内容型网站**的开发需求，比如个人博客、机构官网、新闻门户。这些虽是 Web 1.0、2.0 时代就烂大街的典型 Web-site，但面对不同的甲方，看似雷同的需求往往又有各种细节的定制化。

虽然 WordPress、Discuz!X 等**建站系统**早已非常成熟，但在人们更看重 UI/UX 设计的今天，它们专用的**模板引擎**反而成了二次开发最大的障碍。只要是近两年做过 **Web 前端开发**的同学就知道，不能利用**基于 Node.js 的现代化工具链**的项目，开发起来有多蛋疼。

**Web 前后端分离**催生了 Web 前端岗位，也让 [WordPress][1]、[Ghost][2] 这些老牌 CMS 系统开放了 **RESTful API**。但有的开源项目做得更加激进 —— [Headless CMS][3]，前台界面都省了，只有**后端 API** 和**后台界面**，普通用户看啥你们自己写~

这其中最为优秀的便是 [Strapi][4]，因为它的后台设计不局限于传统的博客、门户，而是：

1. **数据模型**灵活可配置（可用 Git 管理 JSON 配置文件）
2. RESTful API 完全支持 **CRUD**（增删查改）和 **RBAC**（角色权限）
3. 自身是个标准的 **Node.js 后端项目**，并有留有二次开发接口
4. 包括 [Swagger][5] Documentation 在内的插件生态

这使得它既像 BaaS 中的 [LeanCloud][6]，又像 Python 开发框架 [Django][7]，在**简单易用**的同时，又不束缚专业开发者。

{% asset_img Strapi-admin.png %}

## 本地开发

### 创建项目

以下命令会一键**创建项目骨架**、**安装依赖包**、**启动开发模式**：

```shell
npm init strapi-app ~/my-project --quickstart
```

### 开发模式启动

下次继续开发时，则执行以下命令：

```shell
cd ~/my-project
npm run develop
```

如果执行 `npm start`，启动的后台系统只能添加数据，不能修改数据结构，是**线上生产**模式。

### 创建代码库

[在 GitHub 上创建代码库][8]后，把生成的代码传上去：

```shell
git init
git remote add origin https://github.com/my-id/my-project.git
git add .
git commit -m "[add] Strapi framework"
git push
```

## 服务器部署

下面以 Linux (Ubuntu 20.04) 为例，简介线上部署。

### 安装容器环境

```shell
# 更新包管理器数据库
apt update
# 安装 Python PIP
apt install python-pip
# 安装 Docker
curl -fsSL https://get.docker.com | sh
# 安装 Docker Compose
pip install docker-compose
```

### 初始化 Git

为了后面**自动化部署**的方便，需要生成好 **SSH 密钥对**：

```shell
# 生成 SSH Key
ssh-keygen -t rsa -b 4096 -C "my_email@example.com"
# 启动 SSH 后台服务
eval `ssh-agent -s`

cd ~/.ssh
# 添加 SSH Key 私钥
ssh-add id_rsa
# 信任 SSH Key 公钥
cat id_rsa.pub > authorized_keys
```

### 部署配置文件

将以下两个文件放在代码库相应路径中，并提交：

#### Docker 服务编排

`~/my-project/docker-compose.yml`

```yaml
version: '3'

services:
  strapi:
    image: strapi/strapi
    volumes:
      - ./:/srv/app
    ports:
      - '1337:1337'
    environment:
      - NODE_ENV=production
```

以上配置的是最简单的 [SQLite][9] 数据库，小型网站完全足够。

#### GitHub Actions 持续部署

以下配置让我们无需每次更新 Strapi 配置、代码时，登录服务器手动部署新版本：

`~/my-project/.github/workflows/main.yml`

```yaml
name: Deploy Server
on:
  push:
    branches:
      - master
jobs:
  SSH:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@master
      - name: Clean
        run: |
          rm -rf .git/ .gitignore .github/ .editorconfig .eslint*
      - name: Transport
        uses: garygrossgarten/github-action-scp@release
        with:
          local: ./
          remote: ${{ secrets.PATH }}
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          privateKey: ${{ secrets.SSH_KEY }}
      - name: Deploy
        uses: garygrossgarten/github-action-ssh@release
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          privateKey: ${{ secrets.SSH_KEY }}
          command: |
            cd ${{ secrets.PATH }}
            echo BASE_URL=${{ secrets.BASE_URL }} > .env
            docker-compose down
            docker-compose up -d
```

【注意】在推送代码前，务必去 `https://github.com/my-id/my-project/settings/secrets` 页面创建好上述配置中的变量：

|    名称    |                           内容                            |
| :--------: | :-------------------------------------------------------: |
|   `PATH`   |                    服务器**代码目录**                     |
|   `HOST`   |               服务器 **IP 地址**或**域名**                |
|   `USER`   |                     服务器**用户名**                      |
| `SSH_KEY`  | `ssh-keygen` 命令生成的**私钥**，通常存在 `~/.ssh/id_rsa` |
| `BASE_URL` |                  服务器 HTTP 访问根地址                   |

### HTTPS 一键搞定

网站上线时，**服务器配置 HTTPS** 一直是非常难搞的事 ——

- [CloudFlare][10] 域名服务虽自带**免费 HTTPS 证书**，但*墙内速度*一直堪忧
- Nginx 虽有 [Docker 打包的反向代理方案][11]，但配置还是容易出错

幸得网友推荐一神器 —— [Caddy][12]，老子又可以**一把梭**了！

#### 安装 Caddy

```shell
echo "deb [trusted=yes] https://apt.fury.io/caddy/ /" \
    | sudo tee -a /etc/apt/sources.list.d/caddy-fury.list
sudo apt update
sudo apt install caddy
```

#### 启用 Caddy

只要映射好**域名 A 记录**，官网上的一条示例命令就能**自动申请证书并配置好**，`80`、`443` 端口即刻可用：

```shell
caddy reverse-proxy --from example.com --to localhost:1337
```

当然，日常使用肯定得扔后台了：

```shell
sudo -i
nohup caddy reverse-proxy --from example.com --to localhost:1337 > /tmp/caddy.log 2>&1 &
```

## 扩展插件

推荐一些日常项目中常用的**扩展设置**、**外部插件**，让 Strapi 用起来更爽：

- [OAuth 登录](https://strapi.io/documentation/v3.x/plugins/users-permissions.html#providers)
- 文件上传
  - [微软 Azure Storage](https://github.com/jakeFeldman/strapi-provider-upload-azure-storage)
  - [阿里云 OSS](https://github.com/hezzze/strapi-provider-upload-oss)
- [富文本编辑器](https://github.com/TechQuery/strapi-plugin-ckeditor)
- [API 文档生成](https://www.npmjs.com/package/strapi-plugin-documentation)

## 总结

经过前面的一顿折腾，**开发者**只需在本机浏览器中点点鼠标、轻敲键盘，就能实现**网站数据结构**的设计；推送代码到 GitHub，就能实现网站后台的更新。而**运营专员**访问的线上后台锁定了数据结构，他们只能在现有数据表中添加具体数据。这样一来，面对单纯的数据存取，**后端 API** 和**后台 UI** 都不用开发了~

最后，推荐一些后续学习资料：

- [Strapi 后台操作视频](https://strapi.io/blog/release-beta-18-dynamic-zones)
- [「组织管理系统」项目脚手架](https://github.com/kaiyuanshe/OrgServer)
- [Strapi 的 MobX SDK](https://github.com/EasyWebApp/MobX-Strapi)

## 参考文档

1. https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
2. https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script
3. https://yq.aliyun.com/articles/514624
4. https://strapi.io/documentation/v3.x/installation/docker.html
5. https://cgo.gitbook.io/my-it-blog/linux-1/caddy-de-xia-zai-an-zhuang-pei-zhi-qi-dong

[1]: https://wordpress.org/
[2]: https://ghost.org/
[3]: https://github.com/search?q=Headless+CMS&ref=opensearch
[4]: https://strapi.io/
[5]: https://swagger.io/
[6]: https://leancloud.cn/
[7]: https://www.djangoproject.com/
[8]: https://github.com/new
[9]: https://www.sqlite.org/
[10]: https://www.cloudflare.com/
[11]: https://github.com/nginx-proxy/nginx-proxy
[12]: https://caddyserver.com/
