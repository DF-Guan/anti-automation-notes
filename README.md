<div align="center">

# Anti-Automation Notes

**A field guide to anti-automation & verification mechanisms on the modern web.**

*从防御视角盘点：验证码的类型学、验证流程的形态、以及注册滥用应对手段的"攻防对照"笔记。*

**Author · 作者：** [DF-Guan](https://github.com/DF-Guan)

</div>

---

## What is this?

一个面向**防御与研究**的机制笔记仓库。内容聚焦：现代网站在"人与机器区分"这一命题上的工程实践——包括各类验证码的形态与原理、账号验证流程的常见设计模式、以及服务端如何组合这些手段抵御注册滥用。

> ⚠️ 定位说明：本仓库只做**机制观察与原理梳理**，不提供任何可运行的"自动注册/破解验证码"实现。它适合三类读者：
>
> - **产品/安全工程师**：设计注册流程时，了解有哪些可选的验证手段及其权衡
> - **安全研究者**：理解验证码与验证流程的工程细节，作为研究起点
> - **平台运营**：评估自身注册流程的抗滥用强度

## Table of Contents

| # | 文章 | 主题 |
|---|------|------|
| 1 | [CAPTCHA 类型学：从图片到无感挑战](./articles/01-captcha-taxonomy.md) | reCAPTCHA / hCaptcha / Turnstile / 图片验证码 / 无头浏览器指纹的形态与演进 |
| 2 | [验证流程形态学：账号注册的几种"门"](./articles/02-verification-flow-patterns.md) | 邮箱验证码 / magic-link / 手机短信 / 知识问答 / 人工审批的设计模式对照 |
| 3 | [注册滥用防御矩阵：服务端能做些什么](./articles/03-abuse-defense-matrix.md) | 从表单校验到网络策略：抗滥用手段的分层盘点与各自代价 |

## Key Concepts at a Glance

```
Anti-automation layer stack (常见分层):

  L1  Invisible  — 无感挑战: Turnstile / hCaptcha Enterprise / 行为指纹
  L2  Visible    — 有感知验证码: reCAPTCHA v2 / 图片验证码 / 滑动拼图
  L3  Identity   — 账号验证: 邮箱 OTP / magic-link / 短信 / 知识问答 / 人工审批
  L4  Policy     — 策略层: 邮箱域名封锁 / 速率限制 / 网络策略 / IP 信誉
```

## Writing Principles

- **机制优先**：只讲"它是什么、为什么存在、代价几何"，不写"如何绕过"
- **实例匿名**：不点名具体运营方，仅用通用形态描述
- **诚实标注**：每个机制的强弱判断都给出理由与局限，避免"银弹"叙事

## License

CC BY-NC 4.0（署名-非商业性使用）。允许引用学习，禁止用于构建自动化滥用工具。
