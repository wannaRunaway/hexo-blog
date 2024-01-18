---
title: Android-上传module到jitpack.io
tags: Android
---

# 上传步骤

1、根据android官方文档，修改module build.gradle文件

2、上传项目至GitHub。

3、发布release包。

4、到jitpick.io查询当前包的打包状态。

5、打包完成即可implementation。

6、error 新建jitpack.yml文件使用openjdk11。

```
jdk:
  - openjdk11
```



# Build.gradle修改

```java
plugins {
    id 'com.android.library'
    id 'kotlin-android'
    id 'maven-publish'
}
```

```java
afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'com.github.wannaRunaway'
                artifactId = 'networkmanager-retrofit-rxjava'
                version = '1.3'
            }
        }
    }
}
```

看英文就知道意思，你的id，打包项目id，版本号
