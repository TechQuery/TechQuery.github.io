---
title: 脱离 IDE，徒手 Kotlin！
date: 2022-01-06 23:40:06
categories:
  - Development
tags:
  - Java
  - Kotlin
---

Java、PHP 作为中国软件外包行业的扛把子，自然是码农最多、甲方最爱的后端技术栈，我自主创业以来也有不少甲方要求 Java + Spring 的架构。

同时，JSer 一直很反感 Java 庞大的代码量，即便后来大行其道的 [TypeScript][1] 为 [JavaScript][2] 生态引入强大的**类型系统**，但感谢 TS 之父也创造了 [C#][3]，其代码也比 Java 简洁很多。

于是，Java IDE 界新星 IntelliJ IDEA 的东家 JetBrains 发明的 [Kotlin 语言][4]，便因其现代性而受到 [Google Android][5] 和我的青睐~

虽然 Kotlin 官网首页非常小清新地给初学者展示了语言的几大核心用法，但正式文档的入口却繁杂而令人困惑，让我这种觉得他们家 IDE 太重的 [VS Code][6] 铁粉无所适从……

故而有了以下徒手学习之旅 ——

## Hello, Kotlin!

### Just for "fun"

`hello.kt`

```kotlin
fun main() {
    print("Hello, Kotlin!")
}
```

> 参考：https://kotlinlang.org/#try-kotlin

### 安装编译器

2018 年做 Python 教练的经历，给了我[装环境一把梭][7]的底气。

#### Windows

```powershell
choco install kotlinc -y
```

#### Mac OS X

```bash
brew install kotlin
```

### 运行

我只实验成功了“编译为 JAR 包再运行”的方法：

```shell
kotlinc hello.kt -include-runtime -d hello.jar
java -jar hello.jar
```

> 参考：https://www.runoob.com/kotlin/kotlin-command-line.html

## Boot Spring

[Spring Boot][8] 算是 Java 最典范的后端 MVC 框架，再配上 Kotlin，写起来很像 [Node.js + TypeScript][9]、Python + [Flask][10]/Django 等技术栈，利于工程师在它们之间风骚走位，自然值得学习。

### 下载脚手架

```shell
curl https://start.spring.io/starter.zip \
    -d language=kotlin \
    -d type=gradle-project \
    -d dependencies=web,mustache,jpa,h2,devtools \
    -d packageName=com.example.web \
    -d name=Web \
    -o web.zip
```

> 参考：https://spring.io/guides/tutorials/spring-boot-kotlin/#using-command-line

### 安装包管理器

我选择 [Gradle][11] 作为 JVM 语言的包管理器，主要因为它的配置文件不像老牌 Maven 的 XML 那样繁杂，而且支持 **Kotlin 脚本语法**，让我可以全身心地学习 Kotlin。

感谢 [@mythcsj][12] 的解答，让我理解了：

> 对于 JSer 而言，Maven 类似 NPM，既是个包管理器，也有自建包服务器；而 Gradle 类似 [Yarn][13]，是构建在前者成熟包服务器之上的新包管理器。

#### Windows

```powershell
choco install gradle -y
```

#### Mac OS X

```bash
brew install gradle
```

### 墙内镜像

懂的都懂，感谢阿里~

`build.gradle.kts`

```diff
repositories {
	mavenCentral()
+	maven(url = "https://maven.aliyun.com/repository/public")
+	maven(url = "https://maven.aliyun.com/repository/gradle-plugin")
+	maven(url = "https://maven.aliyun.com/repository/spring")
+	maven(url = "https://maven.aliyun.com/repository/spring-plugin")
}
```

> 参考：https://developer.aliyun.com/mvn/guide

### Hello, RESTful!

`src\main\kotlin\com\example\web\HelloController.kt`

```kotlin
package com.example.web;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@RestController
class HelloController {
  @GetMapping("/")
  @ResponseBody
  fun getData(): MutableMap<String, Int> {
    return mutableMapOf("a" to 1)
  }
}
```

> 参考：
>
> 1. https://spring.io/guides/tutorials/spring-boot-kotlin/#_exposing_http_api
> 2. https://spring.io/guides/gs/rest-service/#_create_a_resource_controller

### 运行

感谢 [@aruis][14] 的解答，才让我知道官方脚手架 `HELP.md` 一大堆链接里都没明确提及的本地运行命令是啥……

```shell
./gradlew bootRun
```

### Docker 镜像

再次感谢 [@aruis 发起的 Pull Request][15]，不仅帮我修复了团队脚手架的 bug，还写了 `Dockerfile`：

```dockerfile
FROM openjdk:11.0-jre-slim

ENV JAR_FILE_NAME demo-0.0.1-SNAPSHOT.jar
COPY build/libs/$JAR_FILE_NAME app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

于是，我们可以在传统构建后，再构建 Docker 镜像：

```shell
./gradlew build
docker build .
```

> 参考：https://spring.io/guides/gs/spring-boot-docker/

### GitHub actions + packages

每次手动构建 Docker 镜像很烦，[Docker Hub][16] 官方账号我也总是忘记，那就 GitHub [actions][17] + [packages][18] 服务一把梭吧~

`.github\workflows\release.yml`

```yaml
name: Release Docker image
on:
  push:
    tags:
      - v*
jobs:
  push_to_registries:
    name: Push Docker image to GitHub registry
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: 17
          distribution: adopt
          cache: gradle
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build Kotlin
        run: |
          chmod +x ./gradlew
          ./gradlew build
      - id: ImageName
        uses: ASzc/change-string-case-action@v2
        with:
          string: ${{ github.repository }}
      - name: Push Image
        uses: docker/build-push-action@v2
        with:
          context: .
          platforms: linux/amd64
          push: true
          tags: |
            ghcr.io/${{ steps.ImageName.outputs.lowercase }}:${{ github.ref_name }}
            ghcr.io/${{ steps.ImageName.outputs.lowercase }}:latest
```

每次发版只需用 [Git][19] 打个 tag 即可：

```shell
git tag vX.Y.Z HEAD
git push origin --tags
```

> 参考：
>
> 1. https://github.com/docker/build-push-action/blob/master/docs/advanced/push-multi-registries.md
> 2. https://tomgregory.com/build-gradle-projects-with-github-actions/

### 模板仓库

借助 [GitHub template 仓库][20]这一实用新特性，我们可以轻松创建新项目，最大限度地复用团队沉淀下来的项目公用代码。

以上讲解的所有内容都汇总在我们 [idea2app 团队][21]的脚手架中，欢迎大家试用、反馈：

https://github.com/idea2app/Kotlin-Spring-Boot

## VS Code 扩展

对于近些年风靡 **Web 全栈开发**领域的 VS Code，Java 生态支持自然不能少，虽然以下两个推荐的扩展插件还都是*预览版*，但对于日常应用开发足够了：

1. https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack
2. https://marketplace.visualstudio.com/items?itemName=fwcd.kotlin

> 参考：https://www.jianshu.com/p/90158cdc6d18

## 更多资料

1. Kotlin 常用代码：https://devhints.io/kotlin
2. Kotlin 文档速查：https://devdocs.io/kotlin~1.6/

[1]: https://www.typescriptlang.org/
[2]: https://jscig.github.io/
[3]: https://docs.microsoft.com/en-us/dotnet/csharp/
[4]: https://kotlinlang.org/
[5]: https://www.android.com/
[6]: https://code.visualstudio.com/
[7]: https://tech-query.me/development/coder-start-kit/
[8]: https://spring.io/projects/spring-boot
[9]: https://github.com/idea2app/NodeTS-LeanCloud
[10]: https://flask.palletsprojects.com/
[11]: https://gradle.org/
[12]: https://github.com/mythcsj
[13]: https://yarnpkg.com/
[14]: https://github.com/aruis
[15]: https://github.com/idea2app/Kotlin-Spring-Boot/pull/2
[16]: https://hub.docker.com/
[17]: https://github.com/features/actions
[18]: https://github.com/features/packages
[19]: https://git-scm.com/
[20]: https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository
[21]: https://ideapp.dev/
