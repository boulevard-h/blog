---
title: OpenClaw Cooldown 机制详解
date: 2026-03-02 01:38:00
categories: [Claw🦞]
tags: [OpenClaw, Debugging]
---

在使用 OpenClaw 时，你可能会遇到系统进入 **Cooldown（冷却期）** 的情况。这通常发生在 API 频率限制（Rate Limit）或配额耗尽（Quota Exceeded）时。

### 什么是 Cooldown？

为了保护你的 API 账号不被进一步封禁，OpenClaw 会在检测到特定错误后，在状态文件中自动设置一个 `cooldownUntil` 时间戳。在此期间，OpenClaw 将拒绝尝试调用该 API，以避免触发服务商的更严厉限制。

### 如何手动取消冷却状态？

如果你确定问题已解决（比如已经续费或等待了足够的时间），可以通过以下步骤手动取消冷却：

1. **找到状态文件**：
   文件路径通常为：`~/.openclaw/agents/main/agent/auth-profiles.json`

2. **修改时间戳**：
   * 在 `usageStats` 部分找到对应的服务配置项。
   * 找到 `cooldownUntil` 这一行。
   * 将其值改为 `0` 或直接删除。

### 调整冷却策略

你也可以在 `openclaw.json` 的 `auth.cooldowns` 中全局调整冷却时长：

```json
"auth": {
  "cooldowns": {
    "billingBackoffHours": 1,
    "failureWindowHours": 1
  }
}
```

希望这篇文章能帮你解决“无法调用 API”的问题！🦞
