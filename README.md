<div align="center">

# Anti-Automation Notes

**A field guide to anti-automation & verification mechanisms on the modern web.**
**从防御视角盘点：验证码的类型学、验证流程的形态、以及注册滥用应对手段的"攻防对照"笔记。**

**Author · 作者：** [DF-Guan](https://github.com/DF-Guan)
**License · 许可：** [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

</div>

---

## 🌐 Language · 语言

| 简体中文 | English |
|---|---|
| 点击下方"中文"链接阅读简体中文版 | Read the English version via the links below |

---

## What is this?

一个面向**防御与研究**的机制笔记仓库。内容聚焦：现代网站在"人与机器区分"这一命题上的工程实践——包括各类验证码的形态与原理、账号验证流程的常见设计模式、以及服务端如何组合这些手段抵御注册滥用。

This is a **defense- and research-oriented** note repository about the engineering practices behind "human vs. machine differentiation" on the modern web—CAPTCHA forms and principles, common account-verification flow patterns, and how servers combine these measures against registration abuse.

> ⚠️ **定位说明 / Scope**：本仓库只做**机制观察与原理梳理**，不提供任何可运行的"自动注册/破解验证码"实现；所有核心论断均附经公开检索核验的学术引用。它适合三类读者：
> - **产品/安全工程师**：设计注册流程时，了解有哪些可选的验证手段及其权衡
> - **安全研究者**：理解验证码与验证流程的工程细节，作为研究起点
> - **平台运营**：评估自身注册流程的抗滥用强度
>
> ⚠️ This repo only documents **mechanisms and principles**; it ships **no runnable signup/CAPTCHA-breaking implementation**. Every key claim is backed by publicly-verified academic references.

---

## Table of Contents · 目录

| # | 文章 (Article) | 中文 | English |
|---|---|---|---|
| 1 | CAPTCHA 类型学：从图片到无感挑战<br>(A Taxonomy of CAPTCHAs) | [中文](./articles/01-captcha-taxonomy.md) | [EN](./articles/01-captcha-taxonomy.en.md) |
| 2 | 验证流程形态学：账号注册的几种"门"<br>(A Morphology of Verification Flows) | [中文](./articles/02-verification-flow-patterns.md) | [EN](./articles/02-verification-flow-patterns.en.md) |
| 3 | 注册滥用防御矩阵：服务端能做些什么<br>(An Abuse-Defense Matrix for Registration) | [中文](./articles/03-abuse-defense-matrix.md) | [EN](./articles/03-abuse-defense-matrix.en.md) |

### 文章速览 / Article Abstracts

**No. 1 — CAPTCHA 类型学（A Taxonomy of CAPTCHAs）**

> **摘要 / Abstract**：梳理 CAPTCHA 的形态演进——经典文字/图片验证码、reCAPTCHA v2 与 hCaptcha 等信号评分型、Turnstile 等无感挑战、无头浏览器指纹；逐类分析其识别原理、对真人的负担、对自动化程序的对抗能力与成本天花板。
>
> **关键词 / Keywords**：CAPTCHA；人机区分；设备指纹；无感挑战；反滥用

**No. 2 — 验证流程形态学（A Morphology of Verification Flows）**

> **摘要 / Abstract**：比较即时建号、邮箱 OTP、magic-link、短信验证、知识问答、人工审批等注册**后**验证形态；核心论点是——验证流程的强度取决于**验证对象（邮箱/手机号/人工）的可获得成本**，而非验证码本身。
>
> **关键词 / Keywords**：账号验证；邮箱 OTP；magic-link；临时邮箱；人工审批

**No. 3 — 注册滥用防御矩阵（An Abuse-Defense Matrix for Registration）**

> **摘要 / Abstract**：盘点服务端完整防御分层——入口层、表单/路由层、身份层、策略层、数据层；强调"强度来自组合与纵深"、手段之间存在替代关系，并给出对注册流程强度的合法评估清单。
>
> **关键词 / Keywords**：反滥用；分层防御；速率限制；IP 信誉；临时邮箱；行为风控

---

## Key Concepts at a Glance · 核心概念速览

```
Anti-automation layer stack (常见分层):

  L1  Invisible  — 无感挑战: Turnstile / hCaptcha Enterprise / 行为指纹
  L2  Visible    — 有感知验证码: reCAPTCHA v2 / 图片验证码 / 滑动拼图
  L3  Identity   — 账号验证: 邮箱 OTP / magic-link / 短信 / 知识问答 / 人工审批
  L4  Policy     — 策略层: 邮箱域名封锁 / 速率限制 / 网络策略 / IP 信誉
```

---

## Repository Structure · 仓库结构

```
anti-automation-notes/
├── README.md                          # 本文件：简介、目录、速览、规范
├── LICENSE                            # CC BY-NC 4.0 许可文本
└── articles/
    ├── 01-captcha-taxonomy.md         # 第 1 篇（中文）
    ├── 01-captcha-taxonomy.en.md      # 第 1 篇（English）
    ├── 02-verification-flow-patterns.md       # 第 2 篇（中文）
    ├── 02-verification-flow-patterns.en.md    # 第 2 篇（English）
    ├── 03-abuse-defense-matrix.md             # 第 3 篇（中文）
    └── 03-abuse-defense-matrix.en.md          # 第 3 篇（English）
```

---

## Writing Principles · 写作原则

- **机制优先**：只讲"它是什么、为什么存在、代价几何"，不写"如何绕过"
- **实例匿名**：不点名具体运营方，仅用通用形态描述
- **诚实标注**：每个机制的强弱判断都给出理由与局限，避免"银弹"叙事
- **可追溯**：核心论断附学术文献引用（见各篇末尾"References"），全部经公开检索核验

---

## Citation Note · 引用规范

- 每篇文章独立编号，正文以上标 `[n]` 引用，文末对应 "References / 参考资料" 小节
- 引用格式遵循 **IEEE 参考规范**（作者、题名、会议/期刊、卷期页码、DOI）
- 页码/卷期以原刊为准；文献均经公开检索核验，不作编造

---

## License · 许可

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)（署名-非商业性使用 / Attribution-NonCommercial）。
允许引用学习，禁止用于构建自动化滥用工具。
Citation for learning is allowed; use for building automated-abuse tooling is prohibited.
