# 朝花拾语 · 成语学习 Android APP

「朝花拾语」——国风深色主题的成语学习 Android 应用，与微信小程序 [Idiom-Dictionary-Mini-Program](https://github.com/PerryChenAce/Idiom-Dictionary-Mini-Program) 功能对齐（v1.0.1）。

**最新版下载**：[Releases](https://github.com/PerryChenAce/Idiom-Dictionary-APK/releases)（Android 8.0+，安装时需允许「安装未知来源应用」）

## 功能特性

- **学习**：3D 翻卡学习（成语 / 拼音 / 释义 / 出处 / 典故 / 例句），艾宾浩斯 SRS 复习队列（第 1、2、4、7、15 天复习），收藏星标，当日任务完成可「再学一组」
- **测验**：释义选词 / 看词选义 / 成语接龙三种模式 × 简单 / 中等 / 困难 / 地狱四档难度
  - 难度基于字频生僻度分层：全词库按词中最生僻字的字频序分为常见 / 次常见 / 生僻三层
  - 简单（常见层）、中等（常见+次常见）、困难（全词库 + 形近音近干扰项 + 无拼音）、地狱（仅生僻层 + 约半数为例句挖坑题）
  - 题干不显示例句；答错按学习页详情样式展示释义、出处、背后的故事、例句
  - 作答后点击任意选项，下方卡片可切换查看该成语详解
  - 右滑下一题、左滑回看上一题，实时正确率统计
- **计划**：100 天学习计划（每天约 50 个新词，覆盖全部 5000 词库），当日任务完成自动打卡，连续打卡 streak + 100 格打卡日历，进行中可一键「重新开始」
- **朗读**：TTS 朗读（活泼女声）；「听成语」自动连读模式——逐卡朗读成语与释义并自动标记学会，边听边完成当日学习任务
- **我的**：学习统计、5000 条词库搜索（文字 / 拼音）、分段筛选、生僻度标签、分页加载、一键清空记录
- **数据**：全部学习数据仅存本地（localStorage），零网络请求，不收集任何用户信息

## 技术栈

- **壳**：Capacitor 8（`@capacitor/android`、`@capacitor/app`、`@capacitor/status-bar`、`@capacitor-community/text-to-speech`）
- **前端**：原生 HTML / CSS / JS 单页应用（无框架、无构建步骤），四 Tab 结构
- **词库**：统一词库 5000 条（原 4752 条 + 新增生僻成语 248 条），随包内联；按 Jun Da 现代汉语字频表（9933 字）离线分三层生僻度
- **存储**：`localStorage`，键 `chengyu_progress_v1`，数据模型与小程序一致

## 项目结构

```
chengyu-app/
├── www/                    # H5 应用（Capacitor webDir）
│   ├── index.html          # 页面结构（学习/测验/计划/我的 四视图）
│   ├── app.css             # 样式（墨黑 + 印章红 + 鎏金国风）
│   ├── app.js              # 全部逻辑（SRS/队列/测验/计划/打卡/TTS/听成语）
│   └── data/
│       ├── chengyu.js      # 成语词库（一）
│       ├── chengyu_extra.js# 成语词库（二）
│       ├── chengyu_rare.js # 新增 248 条生僻成语
│       └── tiers.js        # 生僻度分层数据（tools/build_tiers.js 生成）
├── tools/                  # 生僻度分级工具
│   ├── build_tiers.js      # 按字频表生成分层数据
│   ├── tier_overrides.json # 人工校准（脚本重跑不覆盖）
│   └── char-table/         # 字频表与通用规范汉字表
├── android/                # Capacitor 生成的安卓工程
│   ├── app/build.gradle    # 含 release 签名配置（读取 keystore.properties）
│   └── local.properties    # 本机 SDK 路径（不入库）
├── assets/                 # 图标与启动屏源文件（@capacitor/assets 生成）
├── capacitor.config.json   # appId: com.chengyu.app
└── DESIGN.md               # 项目设计文档
```

## 本地构建

前置条件：

- **JDK 21**（Capacitor 8 要求；通过 `-Dorg.gradle.java.home` 指定或设置 `JAVA_HOME`）
- **Android SDK**（在 `android/local.properties` 中配置 `sdk.dir=...`）
- Node.js

```bash
npm install
npx cap sync android

cd android
# debug 包
./gradlew assembleDebug -Dorg.gradle.java.home="C:/path/to/jdk21"
# release 签名包（需 android/keystore.properties，见下）
./gradlew assembleRelease -Dorg.gradle.java.home="C:/path/to/jdk21"
```

产物：`android/app/build/outputs/apk/{debug,release}/`

## GitHub Actions 云端打包

仓库内置 `.github/workflows/build-apk.yml`：每次 push 到 main（或在 Actions 页手动触发）自动执行 `npm ci → cap sync → gradlew assembleDebug`，完成后在对应 workflow run 的 **Artifacts** 中下载 `chengyu-debug-apk`（保留 30 天）。

### release 签名

签名材料不入库。本地放置：

- `android/chengyu-release.keystore`（签名密钥库，**务必离线备份，丢失无法更新 APP**）
- `android/keystore.properties`：

```properties
storeFile=chengyu-release.keystore
keyAlias=chengyu
storePassword=********
keyPassword=********
```

### 图标与启动屏

由 `assets/icon.png`（1024×1024）与 `assets/splash*.png`（2732×2732）生成：

```bash
npx capacitor-assets generate --android
```

## 发布流程

1. `./gradlew assembleRelease` 打出签名包
2. `git tag vX.Y.Z && git push origin vX.Y.Z`
3. GitHub 创建 Release，上传 APK 作为附件

## 文档

- [DESIGN.md](./DESIGN.md)：项目设计文档（技术选型、架构、平台适配、发布渠道、风险）

## License

MIT
