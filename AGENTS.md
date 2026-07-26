# AGENTS

## 项目概览
- Autox.js v7：Android 上支持无障碍自动化的 JavaScript 运行与开发环境，基于 Auto.js 4.1 fork；许可证 MPL-2.0+非商业 + GPLv2（详见 README）。
- 根模块：`app`（主应用与 IDE）、`autojs`（JS 引擎与 API，含 Rhino/Node via Javet）、`automator`（无障碍与控件操作层）、`common`（Compose UI 与工具）、`inrt`（模板运行时，生成 template.apk）、`apkbuilder`（脚本打包签名）、`codeeditor`（内置编辑器资源）、`paddleocr`（OCR 能力）、`mcp`（App 内嵌 MCP 服务端，见 `docs/MCP_USAGE.md`）。
- 版本与 SDK 见 `project-versions.json`：Java 17，compile/target SDK 34，minSdk 27；`settings.gradle.kts` 列出全部模块。

## 构建 & 运行
- 先初始化子模块与依赖：`git submodule update --init --recursive` 获取 `docs/v2`；需要 Node.js 20+（JS API 构建、文档）与 Android SDK/NDK。
- JDK 管理：项目通过 `foojay-resolver-convention` 插件自动下载所需 JDK（17），无需手动配置 `JAVA_HOME`；如需固定可在 `gradle.properties` 设置 `org.gradle.java.home`（注意不要提交机器特定路径）。
- JS API 构建：`./gradlew autojs:buildJsModule`（运行 v6/v7 API Node 构建并拷贝到 assets）；变更 JS 模块后需重跑。
- 模板产物：`app` 构建前会从 `inrt` 模板拷贝 `template.apk` 到 `app/src/main/assets`；若缺失会触发 `buildTemplateApp`。
- 常用命令：
  - Debug 安装：`./gradlew app:buildDebugTemplateApp app:assembleV7Debug app:installV7Debug`
  - Release 包：`./gradlew app:buildTemplateApp app:assembleV7`
  - 仅模板：`./gradlew inrt:assembleTemplateDebug` 或 `inrt:assembleTemplateRelease`
  - 仅 MCP 模块：`./gradlew :mcp:assembleDebug`（运行单测：`./gradlew :mcp:test`）
- 内置代码编辑器资源由 `codeeditor:downloadEditor` 在预构建阶段自动下载（需网络）。

## 测试
- 连接 adb 设备后运行 `./gradlew autojs:assemble`（或在 Android Studio 触发一次构建）。
- 在 `autojs/src/androidTest` 打开测试类并运行；设备端允许安装测试 APK 后继续。
- MCP 协议层单测：`./gradlew :mcp:test`（纯 JVM，覆盖 JSON-RPC initialize/tools-list/tools-call/通知）。

## 文档
- 文档子模块位于 `docs/v2`（git 子模块指向 aiselp/AutoxDoc）。
- 构建离线文档：`./gradlew app:buildDocs`（Node 构建 JS API 文档与 VuePress，然后拷贝到 `app/src/main/assets/docs/v2`）。
- MCP 服务端：`docs/MCP_APP_MCP_SERVER_PLAN.md`（方案）与 `docs/MCP_USAGE.md`（使用与全部工具示例）。

## 其他提示
- UI 主要基于 Compose/Material3，部分旧代码仍依赖 ButterKnife、RxJava2/3；`gradle.properties` 启用了 AndroidX/Jetifier、Gradle 并行与缓存。
- 默认仅生成 arm64-v8a ABI（`app` splits 及 `apkbuilder` 设置），`inrt` template 渠道会剔除模型/资源。
- 大型依赖（Javet、MLKit、ONNX、PaddleOCR 等）在网络受限场景可提前缓存本地 Maven/Gradle。
