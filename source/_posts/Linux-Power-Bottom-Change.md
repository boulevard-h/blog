---
layout: blog
title: Linux 电源键功能修改
date: 2024-07-05 15:22:14
categories: 环境搭建
---

最近有远程办公的需求，而**的 ToDesk 在换了 6.8 Kernel 的 Ubuntu 上老崩溃，所以买了一个米家只能开关，在手机上操控开关。

正常来说强制关机需要长按四五秒的样子，但是这个沟槽的开关只能短按，而不能长按。Ubuntu 20.04 LTS 的短按电源键是弹出一个询问窗口，等待 60s 以后才关机，非常的折磨。所以文本例举和尝试了网上的集中修改按下电源键反应的方法，最终找到了一种由于的方法。

1. 修改 systemd

   ```shell
   sudo nano /etc/systemd/system.conf
   
   #DefaultTimeoutStopSec=90s
   改为
   DefaultTimeoutStopSec=5s
   
   sudo systemctl daemon-reload
   ```

   - 经过测试，无效

2. 修改 gnome settings 的 button-power

   ```shell
   gsettings set org.gnome.settings-daemon.plugins.power button-power 'shutdown'
   或者还有人说
   gsettings set org.gnome.settings-daemon.plugins.power power-button-action 'shutdown'
   ```

   - 尝试了两种方法都没有用，而且我的版本的 gnome 中 power-button-action 根本没有 'shutdown' 这个值

3. 修改 systemd-logind

   ```shell
   sudo nano /etc/systemd/logind.conf
   
   修改如下三行
   PowerKeyIgnoreInhibited=no
   HandlePowerKey=poweroff
   PowerKeyAction=poweroff
   
   sudo systemctl restart systemd-logind
   ```

   - 无效，而且 restart systemd-logind 会重启电脑，吓我一跳

4. 修改 ACPI

   ```shell
   sudo nano /etc/acpi/events/powerbtn
   
   添加
   event=button/power
   action=/sbin/shutdown -h now
   
   sudo systemctl restart acpid
   ```

   - 测试有效，短按电源键直接关机
   - 如果你想重启的话，可以把 `action=/sbin/shutdown -h now` 改为 `action=/sbin/reboot`

