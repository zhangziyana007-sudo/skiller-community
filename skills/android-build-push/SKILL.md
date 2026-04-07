---
name: android-build-push
description: Android 模块编译与 adb 推送流程。当用户需要编译 Android 模块（如 CarSystemUI、HVAC）、adb push 推送 APK 到设备、或重启 SystemUI 时使用此技能。
sub_skills:
  - title: 初始化编译环境
    description: source build/envsetup.sh 并 lunch 指定 target
  - title: 编译模块
    description: 使用 make 命令编译指定模块 (如 CarSystemUI)
    children:
      - title: 后台编译监控
        description: block_until_ms:0 后台执行，监控终端输出判断完成
  - title: ADB 推送 APK
    description: adb root/remount/push 推送编译产物到设备
  - title: 重启 SystemUI
    description: kill 进程 ID 让系统自动拉起新版本
---

# Android 编译与推送

## 编译流程

### 1. 初始化编译环境并编译
```bash
bash -c 'cd <ANDROID_ROOT> && source build/envsetup.sh && lunch <TARGET> && make <MODULE> 2>&1 | tail -30'
```

常用配置：
- ANDROID_ROOT: `/home/ts/projects/Aqirui/lagvm/LINUX/android`
- TARGET: `qssi_au-userdebug`
- MODULE: `CarSystemUI`

注意：必须用 `bash -c` 包裹，避免 zsh 环境冲突导致"缺少操作数"错误。

### 2. 编译耗时约 10-15 分钟
- 使用 `block_until_ms: 0` 将编译放入后台
- 通过读取终端文件监控进度
- 看到 `#### build completed successfully ####` 表示成功

## 推送流程

### 3. adb 推送
```bash
adb root && adb remount && adb push <APK_PATH> <DEVICE_PATH>
```

常用路径映射：
| 模块 | APK 输出路径 | 设备路径 |
|------|-------------|---------|
| CarSystemUI | `out/target/product/qssi_au/system_ext/priv-app/CarSystemUI/CarSystemUI.apk` | `/system_ext/priv-app/CarSystemUI/CarSystemUI.apk` |

### 4. 重启 SystemUI
```bash
PID=$(adb shell pidof com.android.systemui)
adb shell kill $PID
```

SystemUI 会被系统自动重新拉起。

## 注意事项
- 推送前必须通过 AskQuestion 确认用户已准备好设备
- 编译期间使用 interactive_feedback 保持连接
- adb push 可能耗时 30-40 秒（APK 约 70MB）
