---
layout: blog
title: Cursor 免费试用无线续杯教程
date: 2025-02-13 12:28:20
categories: 环境搭建
---

思路：用临时邮箱白嫖cursor的新用户14天免费注册，然而cursor对这种行为有所防范，限制了机器码，所以需要在换邮箱之前修改机器码

用到的工具：

- https://github.com/chengazhen/cursor-auto-free 修改机器码
- https://smailpro.com/ 临时邮箱

## 1. 修改机器码

Windows：

```powershell
irm https://aizaozao.com/accelerate.php/https://raw.githubusercontent.com/yuaotian/go-cursor-help/refs/heads/master/scripts/run/cursor_win_id_modifier.ps1 | iex
```

Mac：

```bash
curl -fsSL https://aizaozao.com/accelerate.php/https://raw.githubusercontent.com/yuaotian/go-cursor-help/refs/heads/master/scripts/run/cursor_mac_id_modifier.sh| sudo bash
```

运行以后，大概会出现：

``` bash
[INFO] 成功修改文件: Cursor.app/Contents/Resources/app/out/vs/code/node/cliProcessMain.js
[INFO] 重新签名应用...
[INFO] 正在关闭 Cursor...
[INFO] 创建应用备份: /Applications/Cursor.backup.20250213_121254.app
[INFO] 安装修改版应用...
[INFO] Cursor 主程序文件修改完成！原版备份在: /Applications/Cursor.backup.20250213_121254.app

[WARN] 是否要修改MAC地址？
0) 否 - 保持默认设置 (默认)
1) 是 - 修改MAC地址
请输入选择 [0-1] (默认 0): [INFO] 已跳过MAC地址修改

[INFO] 文件结构:
/Users/XXX/Library/Application Support/Cursor/User/globalStorage
├── globalStorage
│   ├── storage.json (已修改)
│   └── backups
│       └── storage.json.backup_20250213_121253
│       └── system_id.backup_20250213_121253



[INFO] 正在禁用 Cursor 自动更新...
如果需要恢复自动更新，可以手动删除文件：
/Users/XXX/Library/Application Support/Caches/cursor-updater

[INFO] 成功禁用自动更新

[INFO] 验证方法：
运行命令：ls -l "/Users/XXX/Library/Application Support/Caches/cursor-updater"
确认文件权限显示为：r--r--r--

[INFO] 完成后请重启 Cursor
[INFO] 请重启 Cursor 以应用新的配置
```

## 2. 注册新邮箱

进入 https://smailpro.com/，获得一个临时邮箱，进入 cursor，退出原有账号，然后用临时邮箱注册登录，临时邮箱的验证码会发送到 smailpro 网页上，注册完毕以后，就可以用这个临时邮箱继续白嫖 14 天 free trial 了
