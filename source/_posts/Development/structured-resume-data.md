---
title: 结构化简历数据
date: 2019-08-14 18:14:58
categories:
  - Development
tags:
  - data
  - model
  - document
  - schema
---

参加工作久了，经历公司多了，每次**换工作写简历**都烦得要死 ——

1. 常用招聘网站**更新简历**：简历编辑器改版了，操作习惯大变不说，数据格式很多也对不上号
2. 注册新招聘网站**填简历**：所需内容和之前其它网站差不多，但填法大相径庭

所以，虽说每次只加一项前任的内容，但却*每到一处基本重写*…… 虽说简历是自己脑子里的记忆，但每次向不同相关方传递时，都要用不同的话再描述一遍，削足适履…… 这在**信息时代**是一种*效率极低的沟通方式*！

## 大神说：要有简历规格！

于是，你就能从 GitHub 搜到一个叫 [JSON Resume][1] 的开源项目 —— 除了人人受用的[主题模板][2]和程序员最爱的[命令行工具][3]，一个学术范儿的[标准规范][4]也是必须有的~

了解 Node.JS 的小伙伴马上就能安装 ——

```shell
npm install resume-cli -g
```

然后，反手就新建一份简历 ——

```shell
resume init
```

最后，边改边在浏览器里刷新看效果 ——

```shell
resume serve
```

## 我说：还可以更简单！

有经验的开发者用到这儿可能会问：[能支持 YAML 吗？][5]

很遗憾，**开源界大神**往往像上帝一般，干完最重要的几件事儿就歇着去了，谁哪里用着不爽，自己提交补丁~

但其实也有 **UNIX 哲学式**“各司其职”的折中方案 —— 前置一个 YAML to JSON 转换器（比如 [`yamljs`][6]），就可以用 YAML 写简历了！

### 简历模板生成

```shell
resume init
# 解析简历模板中最深 4 层 JSON 嵌套
json2yaml -d 4 resume.json > resume.yml
```

### 网页版简历导出

```shell
# 输出格式化的 JSON，并清除 resume export 不支持的时间格式
yaml2json resume.yml -p | sed -r s/T[0-9]+:[0-9]+:[0-9]+\.[0-9]+Z//g > resume.json
resume export resume.html
```

### 简历即时预览

先在一个命令行终端启动 **YAML 转换器**监听 ——

```shell
yaml2json resume.yml -s -p -w
```

再在一个命令行终端启动**简历生成器**监听 ——

```shell
resume serve
```

## 一点小坑

1. 没梯子的同学要在 `resume-cli` 安装前启用天朝镜像：

```shell
npm set puppeteer_download_host https://storage.googleapis.com.cnpmjs.org
```

2. 用非默认主题导出时，要先把[主题模板包][7]安装在 `resume-cli` 的对应位置（全局 或 项目）

[1]: https://jsonresume.org/
[2]: https://jsonresume.org/themes/
[3]: https://jsonresume.org/getting-started/
[4]: https://jsonresume.org/schema/
[5]: https://github.com/jsonresume/resume-cli/issues/20
[6]: https://www.npmjs.com/package/yamljs
[7]: https://www.npmjs.com/search?q=jsonresume-theme-*
