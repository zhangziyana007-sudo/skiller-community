---
name: network-troubleshoot
description: 网络问题排查与修复。当用户遇到网络连接问题、代理配置、adb 连接失败、Cursor API 超时、pip/gradle 下载失败时使用此技能。
sub_skills:
  - title: 代理配置检查
    description: 检查系统/终端/IDE 的 HTTP/HTTPS 代理设置
  - title: ADB 连接修复
    description: 排查 adb devices 无设备、unauthorized 等问题
  - title: API 超时排查
    description: 诊断 Cursor API / GitHub API 请求超时
  - title: 包管理器下载修复
    description: 修复 pip/npm/gradle 下载失败问题
    children:
      - title: pip 国内源配置
        description: 配置清华/阿里云镜像源加速下载
      - title: npm registry 切换
        description: 切换到淘宝镜像或其他国内源
---

# 网络问题排查

## 诊断步骤

### 1. 基本连通性
```bash
ping -c 3 8.8.8.8          # 网络是否通
ping -c 3 google.com        # DNS 是否正常
curl -I https://www.baidu.com  # HTTP 是否通
```

### 2. 代理检查
```bash
echo $http_proxy $https_proxy $all_proxy
env | grep -i proxy
```

### 3. DNS 检查
```bash
nslookup google.com
cat /etc/resolv.conf
```

## 常见问题修复

### adb 连接失败
```bash
adb kill-server && adb start-server
adb devices                  # 检查设备列表
adb reconnect               # 重连
```

如果设备未授权：
```bash
adb kill-server
rm ~/.android/adbkey*        # 清除旧密钥
adb start-server             # 重新授权
```

### pip 安装超时
使用国内镜像源：
```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <package>
```

永久配置：
```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### Gradle 下载慢
在 `~/.gradle/gradle.properties` 中添加代理：
```properties
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=7890
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=7890
```

### Cursor API 连接问题
1. 检查代理设置：`Ctrl+Shift+P` → `Preferences: Open User Settings (JSON)`
2. 确认 `http.proxy` 配置正确
3. 设置 `"http.proxyStrictSSL": false` 排除 SSL 问题
4. 测试连通性：`curl https://api2.cursor.sh`

## Cursor 设置中的网络配置
```json
{
  "http.proxy": "http://127.0.0.1:7890",
  "http.proxyStrictSSL": false,
  "http.proxySupport": "on"
}
```

## 注意事项
- 修改代理配置后需重启 Cursor
- adb 通过 USB 连接不受网络代理影响
- 企业网络可能有防火墙限制，联系 IT 部门确认
