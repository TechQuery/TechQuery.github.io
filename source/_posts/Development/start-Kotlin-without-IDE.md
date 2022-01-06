---
title: 脱离 IDE，徒手 Kotlin！
date: 2022-01-06 23:40:06
categories:
  - Development
tags:
  - Java
  - Kotlin
---

## Hello, Kotlin!

### Just for "fun"

`hello.kt`

```kotlin
fun main() {
    print("Hello, Kotlin!")
}
```

此处参考了 https://kotlinlang.org/#try-kotlin

### 安装编译器

https://tech-query.me/development/coder-start-kit/

#### Windows

```powershell
choco install kotlinc -y
```

#### Mac OS X

```bash
brew install kotlin
```

### 运行

```shell
kotlinc hello.kt -include-runtime -d hello.jar
java -jar hello.jar
```

此处参考了 https://www.runoob.com/kotlin/kotlin-command-line.html

## Boot Spring

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

此处参考了 https://spring.io/guides/tutorials/spring-boot-kotlin/#using-command-line

### 安装包管理器

#### Windows

```powershell
choco install gradle -y
```

#### Mac OS X

```bash
brew install gradle
```

### 墙内镜像

```diff
repositories {
	mavenCentral()
+	maven(url = "https://maven.aliyun.com/repository/public")
+	maven(url = "https://maven.aliyun.com/repository/gradle-plugin")
+	maven(url = "https://maven.aliyun.com/repository/spring")
+	maven(url = "https://maven.aliyun.com/repository/spring-plugin")
}
```

此处参考了 https://developer.aliyun.com/mvn/guide

### 运行

```shell
gradle build
java -jar build/libs/demo-0.0.1-SNAPSHOT.jar
```

此处参考了 https://spring.io/guides/gs/rest-service/#_build_an_executable_jar

### Hello, RESTful!

`src\main\kotlin\com\example\web\HttpControllers.kt`

```kotlin
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;

@RestController
class HelloController {
  @GetMapping("/")
  fun foo(): MutableMap<String, Int> {
    return mutableMapOf("a" to 1)
  }
}
```

此处参考了：

1. https://spring.io/guides/tutorials/spring-boot-kotlin/#_exposing_http_api
2. https://spring.io/guides/gs/rest-service/#_create_a_resource_controller

## 更多资料

1. Kotlin 常用代码：https://devhints.io/kotlin
2. Kotlin 文档速查：https://devdocs.io/kotlin~1.6/
