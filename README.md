# Drama SDK 二进制发布仓库

此目录只发布 `drama-sdk.aar`，`drama-sdk.pom` 声明其公共依赖；不需要 Android SDK 或原源码工程。

AAR 的构建方式为：保留原源码目录，由唯一的 `business/drama-sdk` module 编译当前依赖闭包的 23 个目录源码一次，不使用旧 module AAR。
生成目录只用于命名空间、资源和 Manifest 适配；旧 Gradle project 仅用于读取依赖声明，不参与编译。
四个尚缺源码的私有库（pikachu-runtime、network、client-monitor、hive）从当前声明仓库或 Gradle 缓存解析 AAR，提取 `classes.jar` 后与源码统一执行 R8，其公共传递依赖保留在 POM。
`DramaSDK` 及公开回调保留名称和 Kotlin Metadata。此发布仓库只接收上述构建的最终 AAR 与 POM，不执行这些源码编译步骤。

本地验证（不会写入 Maven Local）：

```powershell
.\gradlew.bat publishDramaSdkPublicationToVerificationRepository
```

产物在 `build/repo/`。默认坐标 `com.dz.sdk:drama-sdk:1.0.0`。
可使用 `-Pgroup=... -Partifact=... -Pversion=...` 覆盖；环境变量 `GROUP`、`ARTIFACT`、`VERSION` 优先。

将本目录上传至自己的 GitHub 仓库，创建版本 Tag，再到 https://jitpack.io 输入仓库 URL 并构建该 Tag。
JitPack 提供坐标环境变量，本脚本使用这些坐标执行 `publishToMavenLocal`。
宿主添加 JitPack 及公共依赖所需仓库后，使用 JitPack 页面显示的准确依赖坐标。

宿主 Gradle 示例（Groovy）：

```groovy
// settings.gradle 的 dependencyResolutionManagement.repositories
repositories {
    google()
    mavenCentral()
    maven { url 'https://maven.aliyun.com/repository/releases/' }
    maven { url 'https://jitpack.io' }
}

// app/build.gradle
dependencies {
    implementation 'com.github.用户名:仓库名:Tag'
}
```

以 JitPack 页面实际返回的坐标为准。当前播放器依赖
`com.aliyun.sdk.android:AliyunPlayer:7.16.0-full-57550499` 需要上述阿里云 releases 仓库，不能只配置 Maven Central。
必须通过 Maven 坐标同时消费 AAR 与 POM；不要仅用 `fileTree` 或 `files('drama-sdk.aar')` 引入后忽略 POM，否则公共依赖不会自动补齐。

不要上传源码工程、签名文件、`mapping.txt`、旧模块产物或依赖缓存。本仓库不生成 sources/javadoc 制品。
版本变更时需同时替换 AAR 与 POM，再创建新的 Tag；不要覆盖已发布 Tag。
