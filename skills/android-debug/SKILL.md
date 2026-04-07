---
name: android-debug
description: Android 调试日志添加与分析流程。当用户需要添加调试日志、抓取 logcat、分析 bug 日志、定位 Android 系统问题时使用此技能。
sub_skills:
  - title: 添加调试日志
    description: 在 Java/Kotlin 代码中添加 Log.d/e 输出
  - title: 抓取 logcat
    description: 使用 adb logcat 过滤指定 TAG 获取运行日志
  - title: 分析 bug 日志
    description: 解析 crash log / ANR trace / tombstone 定位问题
  - title: 定位系统问题
    description: 追踪 SystemUI / Framework 层的运行异常
    children:
      - title: 进程状态检查
        description: 检查进程是否存活、内存占用、线程状态
      - title: 服务绑定追踪
        description: 追踪 Service 绑定/解绑的生命周期
---

# Android 调试与日志分析

## 添加调试日志

### 日志工具
项目使用自定义 `LogUtil` 而非标准 `android.util.Log`：
```java
LogUtil.d(TAG, "message");   // Debug
LogUtil.w(TAG, "message");   // Warning
LogUtil.e(TAG, "message");   // Error
```

### 日志规范
- 使用统一前缀方便过滤，如 `[DEBUG_TEMP]`、`[DEBUG_HVAC]`
- 记录关键变量的前后值变化
- 调试完成后必须清理所有临时日志

## 抓取日志

### 实时抓取
```bash
adb logcat -s TAG_NAME:D | tee /tmp/debug_log.txt
```

### 带时间范围抓取
```bash
adb logcat -v time -d > /tmp/full_log.txt
```

### 按关键字过滤
```bash
adb logcat -d | grep "关键字"
```

## 分析流程

1. **定位异常**：搜索 Exception、Error、crash 关键字
2. **追溯调用链**：根据时间戳追踪事件顺序
3. **对比正常/异常**：正常操作和异常操作的日志差异
4. **确认根因**：找到第一个异常点

## 常用调试命令

| 操作 | 命令 |
|------|------|
| 查看进程是否存活 | `adb shell pidof <package>` |
| 查看全局设置 | `adb shell settings get global <key>` |
| 写入全局设置 | `adb shell settings put global <key> <value>` |
| 强杀进程 | `adb shell kill <pid>` |
| 清除应用数据 | `adb shell pm clear <package>` |

## 注意事项
- 添加日志后需重新编译推送（参考 android-build-push 技能）
- 日志分析完毕后，确认根因前先通过 AskQuestion 与用户沟通
- 修复后必须清理所有调试日志再编译最终版本
