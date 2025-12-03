# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

TaskbarLyricsPlugin 是一个适用于 Salt Player For Windows (SPW) 的任务栏歌词插件，支持 Steam 版和微软商店版。这是一个基于 Kotlin/JVM 开发的插件，使用 PF4J 插件框架。

## 构建和开发命令

### 构建插件
```bash
# 编译插件
./gradlew plugin

# 编译为 JAR 文件
./gradlew build
```

### 测试和运行
```bash
# 运行测试（如果有）
./gradlew test

# 清理构建产物
./gradlew clean
```

## 项目架构

### 核心组件

1. **TaskbarLyricsPlugin.kt** - 主插件类
   - 继承自 `Plugin`，负责插件的启动和停止
   - 初始化 HTTP 服务器、SMTC 控制器和配置管理器
   - 插件生命周期管理

2. **HttpServer.kt** - HTTP API 服务器
   - 运行在端口 35374
   - 提供播放控制、歌词获取、配置管理等 REST API
   - 支持多种歌词源：网易云、QQ音乐、酷狗、本地文件等
   - 使用 Jetty 服务器实现

3. **SmtcController.kt** - 系统媒体传输控制
   - 处理媒体键事件（播放/暂停、上一曲、下一曲等）
   - 使用 JNA 与 Windows API 交互
   - 创建隐藏窗口接收系统媒体键消息

4. **PlaybackExtension.kt** - 播放扩展点
   - 实现 SPW 的 `PlaybackExtensionPoint` 接口
   - 监听播放状态变化和歌词更新
   - 从网络获取歌曲封面信息

5. **PlaybackStateHolder.kt** - 播放状态管理
   - 维护当前播放状态的单例对象
   - 管理歌词缓存和时间同步
   - 线程安全的状态管理

6. **ConfigManager.kt** - 配置管理
   - 管理插件配置（字体、颜色、对齐方式等）
   - 支持从文件和 SPW 配置系统加载配置
   - 提供配置变更监听机制

### 配置系统

插件配置定义在 `src/main/resources/preference_config.json` 中，包含：
- 字体设置（字体家族、大小、颜色）
- 显示设置（背景颜色、对齐方式、是否显示翻译）
- 操作按钮（应用配置、重置为默认值）

配置文件位置：`%APPDATA%/Salt Player for Windows/workshop/data/TaskbarLyricsPlugin/config.json`

### API 端点

HTTP 服务器提供的主要端点：

- `/api/now-playing` - 获取当前播放状态
- `/api/play-pause` - 播放/暂停控制
- `/api/next-track` - 下一曲
- `/api/previous-track` - 上一曲
- `/api/volume/up` - 音量增加
- `/api/volume/down` - 音量减少
- `/api/mute` - 静音切换
- `/api/lyric163` - 获取网易云歌词
- `/api/lyricqq` - 获取QQ音乐歌词
- `/api/lyrickugou` - 获取酷狗歌词
- `/api/lyric` - 从音频文件元数据获取歌词
- `/api/lyricfile` - 从本地 LRC 文件获取歌词
- `/api/lyricspw` - 获取 SPW 内部歌词
- `/api/pic` - 获取歌曲封面
- `/api/config` - 获取插件配置

## 依赖项

主要依赖项包括：
- **SPW Workshop API** (0.1.0-dev14) - Salt Player 插件开发框架
- **Jetty** (11.0.15) - HTTP 服务器
- **JNA** (5.10.0) - Java Native Access，用于调用 Windows API
- **Gson** (2.10.1) - JSON 处理
- **JAudioTagger** (3.0.1) - 音频文件元数据读取

## 开发注意事项

### 插件开发规范
- 插件需要实现 `Plugin` 类并配置正确的元数据
- 使用 `@Extension` 注解标记扩展点实现
- 插件 ID: `TaskbarLyricsPlugin`
- 版本: `2.0.0`

### 线程安全
- `PlaybackStateHolder` 使用 `@Volatile` 和 `synchronized` 确保线程安全
- HTTP 请求处理和播放状态更新需要考虑并发访问
- 歌词缓存使用 `ConcurrentHashMap` 管理

### Windows API 集成
- 使用 JNA 调用 Windows API 处理媒体键
- 创建隐藏窗口接收系统消息
- 处理 `WM_APPCOMMAND` 消息

### 错误处理
- 所有外部 API 调用都有异常处理
- 网络请求设置合理的超时时间
- 配置加载失败时使用默认值

## 调试和日志

- 使用 `println` 输出调试信息（在实际部署中建议替换为正式的日志框架）
- HTTP 请求和响应都有详细的错误日志
- 配置变更和播放状态变化都有日志记录

## 插件打包

生成的插件包格式：`plugin-TaskbarLyricsPlugin-2.0.0.zip`
- 包含编译后的类文件和所有依赖项
- 插件清单包含必要的元数据
- 可通过 SPW 的模组管理系统导入

## 相关资源

- [TaskbarLyricsPlugin-resources](https://github.com/zmxlsss666/TaskbarLyricsPlugin-resources) - 使用的核心组件
- [SPW Workshop API 文档] - 插件开发框架文档