---
layout: blog
title: Cursor 免费试用无线续杯教程
date: 2025-02-13 12:28:20
categories: 环境搭建
---

思路：用临时邮箱白嫖cursor的新用户14天免费注册，然而cursor对这种行为有所防范，限制了机器码，所以需要在换邮箱之前修改机器码

用到的工具：

- https://github.com/chengazhen/cursor-auto-free 修改机器码
- https://activity.adspower.net/ 指纹浏览器
- ~~https://smailpro.com/ 临时邮箱~~

## 1. 修改机器码

Windows：

```powershell
irm https://raw.githubusercontent.com/yuaotian/go-cursor-help/refs/heads/master/scripts/run/cursor_win_id_modifier.ps1 | iex
```

Mac/Linux：

```bash
curl -fsSL https://raw.githubusercontent.com/yuaotian/go-cursor-help/refs/heads/master/scripts/run/cursor_mac_id_modifier.sh -x http://127.0.0.1:7897 | sudo bash
```

这里 `-x` 选项是系统代理软件的地址，自行修改或删除

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

之前使用smailpro等临时邮箱网站来白嫖14天免费试用，最近发现可能cursor的风控有所加强，这些网站的域名被ban了，申请试用的时候提示unauthorized request

目前暂时比较好的解决办法是，下载一个AdsPower指纹浏览器，然后注册一个微软outlook邮箱，被ban的风险稍微小一些
