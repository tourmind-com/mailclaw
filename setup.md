# MailClaw Setup

> OpenClaw 在用户完成 Gmail 授权后读取此文件，注册 heartbeat 定时任务。

## 触发条件

用户首次完成 Gmail OAuth 授权（`GET /auth/status?app=gmail` 返回 `connected: true`）后，注册以下 heartbeat 任务。

## Heartbeat 任务

<!-- openclaw:heartbeat
tasks:
  - name: daily-digest-morning
    interval: 24h
    time: "09:00"
    prompt: "Read {baseDir}/heartbeat.md and follow every step exactly."

  - name: daily-digest-evening
    interval: 24h
    time: "17:00"
    prompt: "Read {baseDir}/heartbeat.md and follow every step exactly."
-->

| Task | Time | 用途 |
|------|------|------|
| `daily-digest-morning` | 09:00 | 早间邮件简报 |
| `daily-digest-evening` | 17:00 | 晚间邮件简报 |

## 简报格式

Heartbeat 执行后输出的简报格式参考 `heartbeat.md`，示例：

```
☀️ Email Digest · Apr 8

3 emails pending:
• Sarah Lee: Asking about next week's schedule
• GitHub: PR #142 awaiting review
• Product Hunt: Daily featured picks

[→ Open processing page] (link valid for 24h)
🔑 Verification code: 847291
```
