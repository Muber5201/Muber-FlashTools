# Muber Flash Tools 1.2.0 Beta · 源码

基于 `dd` 的 Android 分区镜像刷写工具，需要超级用户权限（KernelSU / Magisk / APatch）。

界面使用 **KernelSU-Style-UI-Kit** 的 Miuix（Jetpack Compose）组件，并额外提供一套
Material 3 界面，可在应用「设置 → 外观」里切换；支持主题色、深浅色、浮动胶囊底栏、
分区搜索、刷写进度、备份与 SHA-256 回读校验。

## 环境要求

| 项目 | 版本 |
| --- | --- |
| JDK | 21 |
| Gradle | 9.7.1（仓库自带 gradlew） |
| Android Gradle Plugin | 9.4.1 |
| Kotlin / Compose 编译器 | 2.4.20 |
| compileSdk | 37.2 |
| buildTools | 37.0.0 |
| minSdk / targetSdk | 31 / 36 |

> 注意：Compose / Material3 依赖要求 compileSdk ≥ 37，必须安装 `platforms;android-37.2`
> （该平台目前位于 SDK Channel 3，Android Studio 稳定版可能看不到）。

### 安装 SDK 组件

```bash
sdkmanager --channel=3 "platforms;android-37.2" "build-tools;37.0.0" "platform-tools"
```

## 构建

Android Studio：直接 `File → Open` 打开本目录，等待 Gradle 同步后运行 `app`。

命令行：

```bash
# 首次需要告诉 Gradle SDK 位置（Android Studio 会自动生成 local.properties）
echo "sdk.dir=/path/to/Android/sdk" > local.properties

./gradlew :app:assembleDebug      # 产物 app/build/outputs/apk/debug/app-debug.apk
./gradlew :app:assembleRelease    # 产物 app/build/outputs/apk/release/
```

签名：

- 根目录存在 `keystore.jks` 时，release 会用其中的 `muber` 别名签名（口令见 app/build.gradle.kts）
- 没有 keystore 时 release 产物未签名，可自行 `apksigner` 签名，或改用 `assembleDebug`
- 想覆盖安装到别人已装的版本，必须使用**同一个签名**；本项目发布用的 keystore 不随源码提供

## 目录结构

```
app/src/main/java/com/muber/flashtools/
├── MainActivity.kt     入口、状态、文件选择
├── Settings.kt         设置项与 SharedPreferences 持久化
├── Theme.kt            Miuix / Material 主题切换
├── MiuixScreens.kt     Miuix 风格：首页 / 刷写 / 分区 / 设置
├── MaterialScreens.kt  Material 3 风格：同上
├── Shell.kt            su 会话、分区枚举、dd 脚本协议
├── FlashLogic.kt       复制镜像、备份、刷写、校验、重启
└── State.kt            运行时状态
```

## 常见问题

- **报 compileSdk 37 / AAR metadata 错误**：确认已安装 `platforms;android-37.2`，并且
  `app/build.gradle.kts` 中是 `compileSdk { version = release(37) { minorApiLevel = 2 } }`。
- **依赖下载慢**：`settings.gradle.kts` 已把阿里云镜像放在最前，可按需替换。
- **在 arm64 Linux / 手机 proot 里构建**：AGP 自带的 aapt2 只有 x86_64 版本，
  需要 `android.aapt2FromMavenOverride` 指向 arm64 可用的 aapt2，见 `BUILD.md`。
  源码在上方压缩包中
