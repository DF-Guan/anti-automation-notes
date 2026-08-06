# CAPTCHA 类型学：从图片到无感挑战

> **编号**：No. 1 · **作者**：[DF-Guan](https://github.com/DF-Guan) · **语言**：[English](./01-captcha-taxonomy.en.md) | 中文
> **许可**：CC BY-NC 4.0 · **定位**：防御与研究视角的机制笔记，不含任何可运行实现

---

## 摘要

CAPTCHA（Completely Automated Public Turing test to tell Computers and Humans Apart）是现代 Web 上区分"真人"与"程序"的基础设施。本文以防御视角梳理其形态演进：从经典文字/图片验证码，到 reCAPTCHA v2、hCaptcha 等信号评分型方案，再到 Turnstile 等无感挑战与无头浏览器指纹。通过文献与工程实践两条线索，逐类分析其识别原理、对真人的负担、对自动化程序的对抗能力，以及各自的成本天花板。

**关键词**：CAPTCHA；人机区分；设备指纹；无感挑战；反滥用

---

## 1. 引言：为什么需要 CAPTCHA

CAPTCHA 的存在意义只有一个：**在无需人工审计的前提下，自动区分"真人"与"程序"**。注册、登录、评论、投票等场景一旦可以被程序批量驱动，就会出现垃圾内容、虚假账号、刷量等问题。

「自动图灵测试」这一概念由 von Ahn 等人在 2003 年正式提出<sup>[1]</sup>：一个合格的 CAPTCHA 必须满足——计算机程序能够自动生成并评判测试；几乎所有人类都能通过；而同时代的计算机程序几乎无法通过。2004 年，他们又在《Communications of the ACM》上给出了更完整的阐述<sup>[2]</sup>。

所有 CAPTCHA 本质上都是在做一个**代价博弈**：

- 对真人：验证成本要**足够低**，否则用户流失
- 对程序：识别/绕过成本要**足够高**，否则形同虚设
- 对运营方：接入成本、误伤率、合规风险都要可控

这个博弈决定了 CAPTCHA 的整个演进方向——**从"给人出题"逐步转向"对程序做检查"**。

---

## 2. 类型谱系

```
                给"人"出题 ────────────────► 给"程序"做检查
  图片文字    图片选择   滑块拼图   行为验证   设备指纹   无感挑战
  高感知/高成本         ──►               低感知/低成本
```

### 2.1 经典文字/图片验证码（Classic text/image CAPTCHA）

最早形态：扭曲字符 + 干扰线，要求输入。后来衍生出**图片分类**（选出路牌/斑马线/红绿灯）。

**特征**
- 有独立的图片资源或输入框，字段名常含 `captcha`
- 需要一次真实的视觉交互

**代价**
- 对真人：干扰严重，尤其对可访问性不友好
- 对运营方：OCR/深度学习模型对扭曲文字破解率高，属于"劳动密集型但不保险"的手段。其历史背景是：早期 OCR 对老旧印刷品（褪色、泛黄）无法识别约 20% 的单词，von Ahn 等人于是把「人脑识别」打包成众包——人验证的同时顺便完成古籍数字化，即 reCAPTCHA<sup>[3]</sup>
- 现状：已基本退出主流商业站点，多残留在老旧论坛系统（如一些传统 BBS 自建题）中

### 2.2 reCAPTCHA v2（"I'm not a robot" 复选框）

点一下复选框，多数情况下即通过。原理是**在点击前后采集大量浏览器交互信号**（鼠标轨迹、点击历史、cookies、页面行为），交给后端模型打分。

**特征**
- 可见的复选框 + 品牌水印
- 信号不足时会升级出**图片选择题**（"选中所有包含红绿灯的格子"）

**代价**
- 对真人：很低（多数情况一下点击）
- 对程序：可通过大量预渲染模拟交互信号绕过——**它防的是"机械操作"，防不住"模拟真人"**
- 局限：依赖服务端模型质量，且每年都被攻防拉锯。Bursztein 等人在 CHI 2014 的实测显示，为 Google 设计的更易用验证码人类通过率可达 95.3%，并明确指出**验证码不应被孤立使用**，而应作为整体反滥用系统的一部分<sup>[4]</sup>

### 2.3 hCaptcha

与 reCAPTCHA 同类，差异在商业模式与形态：以"人机标注数据"为核心卖点，图片选择题的占比更高，样式可深度定制（企业版支持无感模式）。

**特征**
- 同样有复选框/图片题形态
- 部分站点把 hCaptcha 做成**提交时才加载**——即"表单全填完、一点提交才冒出来"，显著增加自动化的挫败感

**代价**
- 对真人：图片题多时体验下降
- 对程序：与 reCAPTCHA 同属"信号评分制"，强度取决于厂商模型

### 2.4 Turnstile（Cloudflare）

"无感挑战"的代表：页面上**没有任何可见控件**，后台自动评估请求可信度，可信则直接发 token，不可信才升级出交互题。

**特征**
- 常表现为一个隐藏字段 `cf-turnstile-response`，页面无可见元素
- 依赖**出口 IP 信誉**：同一 IP 若已被标记（例如触发过大量挑战），token 可能长时间为空、不生成
- 强绑定 Cloudflare 全套防护（WAF、代理、CDN）

**代价**
- 对真人：近乎零
- 对程序：**IP 信誉成为决定性变量**——换一个干净出口往往直接通过；也因此对共享出口/代理环境很不友好（会大面积误伤）

> 观察点：Turnstile 的"无感"是靠**把成本转移到网络层与信誉层**。它不跟程序"斗智"，而是跟"出口环境"斗——这是它与 reCAPTCHA 类最本质的区别。

### 2.5 无头/指纹类挑战（headless & fingerprint checks）

严格说不算 CAPTCHA，而是"JS 挑战 + 指纹采集"：

- 页面先执行一段 JS（如计算一个数学任务、渲染 canvas），同时采集 `navigator`、`screen`、字体、Canvas 指纹
- 特征不足或不一致（如无头浏览器、禁用 JS、WebGL 渲染异常）→ 判为可疑
- 常见形态：`Performing security verification…` 加载动画、空白页、`NS_ERROR_NET_RESET`

**代价**
- 对真人：透明无感
- 对程序：专门针对"自动化浏览器"的已知特征，需要持续对抗检测规则更新
- 指纹的数学基础：Eckersley 在 2010 年的研究给出浏览器指纹约 18.1 bits 的信息熵，带 Flash/Java 的环境中约 94.2% 的浏览器指纹唯一<sup>[5]</sup>——这说明「设备指纹」确有区分度，但它的"唯一性"同样意味着对隐私浏览器、企业网络、公共 IP 用户的高误伤与隐私代价

---

## 3. 一个实用视角：如何"识别"页面上是哪类验证码

对研究/运维场景，能从页面结构快速判断验证码类型是很有用的。以下是**纯静态观察**的常见信号（不涉及任何绕过）：

| 类型 | 静态信号 |
|---|---|
| Turnstile | 隐藏字段 `cf-turnstile-response`；URL 含 `challenges.cloudflare.com/turnstile` |
| reCAPTCHA | `g-recaptcha-response`；`.g-recaptcha` 容器；frame 含 `recaptcha` |
| hCaptcha | frame 含 `hcaptcha`；`[class*='hcaptcha']` |
| 图片验证码 | 输入框 name/class 含 `captcha` + 页面出现图片资源 |
| 指纹挑战 | 页面无表单、只有加载动画/空白；网络层直接重置 |

---

## 4. 演进规律总结

1. **从"出题"到"检查"**：让真人做题的体验成本太高，商业站点全面转向"对程序做行为/环境评分"。这背后有硬数据支撑：Bursztein 等人提出，一旦自动方案成功率超过约 1%，该验证码即视为实际不安全<sup>[6]</sup>；他们的 WOOT 2014 研究进一步显示，基于文字的经典验证码已接近尽头——在不调整参数的情况下，通用机器学习方案即可在 Yahoo 达到 5.33%、ReCaptcha 22.67%、eBay 51.39%、CNN 51.09% 的破解成功率，全部显著超过 1% 阈值<sup>[7]</sup>。音频验证码同样脆弱：专用系统可破解微软约 48.9%、雅虎约 45.5% 的音频验证码，且往往比人类更准<sup>[8]</sup>
2. **信誉成为新战场**：IP 信誉、设备信誉、账号历史信誉，正在替代"一道题"的作用
3. **多因子组合 > 单点**：强站点往往"表单校验 + 无感挑战 + 后续验证流程"层层叠加（见第 2 篇）
4. **没有银弹**：每一类都有各自的成本天花板——感知型伤体验、无感型依赖信誉、指纹型依赖特征库——这就是它们至今共存的原因

## 5. 延伸思考

- **可访问性与合规**：纯视觉 CAPTCHA 对残障用户是障碍；WCAG 与 GDPR/CCPA 都在倒逼厂商提供无障碍与隐私替代方案——这也是 Turnstile 这类"无感方案"被更多站点接纳的原因之一。
- **成本不对称**：验证码厂商是"为全人类发题"，防御方永远在追赶；理解这一点，才会明白为什么"验证码+验证流程+策略"必须组合使用。

---

## 参考资料

1. L. von Ahn, M. Blum, N. J. Hopper, and J. Langford, "CAPTCHA: Using hard AI problems for security," in *Advances in Cryptology — EUROCRYPT 2003*, LNCS 2656, Springer, 2003, pp. 294–311. DOI: [10.1007/3-540-39200-9_18](https://doi.org/10.1007/3-540-39200-9_18)
2. L. von Ahn, M. Blum, and J. Langford, "Telling humans and computers apart automatically," *Communications of the ACM*, vol. 47, no. 2, pp. 56–60, 2004. DOI: [10.1145/966389.966390](https://doi.org/10.1145/966389.966390)
3. L. von Ahn, B. Maurer, C. McMillen, D. Abraham, and M. Blum, "reCAPTCHA: Human-based character recognition via web security measures," *Science*, vol. 322, no. 5898, pp. 1465–1468, 2008. DOI: [10.1126/science.1160379](https://doi.org/10.1126/science.1160379)
4. E. Bursztein, A. Moscicki, C. Fabry, S. Bethard, D. Jurafsky, and J. C. Mitchell, "Easy does it: More usable CAPTCHAs," in *Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI 2014)*, ACM, 2014, pp. 2637–2646. DOI: [10.1145/2556288.2557322](https://doi.org/10.1145/2556288.2557322)
5. P. Eckersley, "How unique is your web browser?," in *Privacy Enhancing Technologies (PETS 2010)*, LNCS 6205, Springer, 2010, pp. 1–18. DOI: [10.1007/978-3-642-14527-8_1](https://doi.org/10.1007/978-3-642-14527-8_1)
6. E. Bursztein, M. Martin, and J. C. Mitchell, "Text-based CAPTCHA strengths and weaknesses," in *Proceedings of the 18th ACM Conference on Computer and Communications Security (CCS 2011)*, ACM, 2011, pp. 125–138. DOI: [10.1145/2046707.2046724](https://doi.org/10.1145/2046707.2046724)
7. E. Bursztein, J. Aigrain, A. Moscicki, and J. C. Mitchell, "The end is nigh: Generic solving of text-based CAPTCHAs," in *8th USENIX Workshop on Offensive Technologies (WOOT 2014)*, USENIX, 2014.
8. E. Bursztein, R. Beauxis, H. Paskov, D. Perito, C. Fabry, and J. Mitchell, "The failure of noise-based non-continuous audio CAPTCHAs," in *IEEE Symposium on Security and Privacy (S&P 2011)*, IEEE, 2011, pp. 19–31. DOI: [10.1109/SP.2011.14](https://doi.org/10.1109/SP.2011.14)

> 说明：以上文献均经公开检索核验。引用格式遵循 IEEE 参考规范；页码、卷期以原刊为准。本文仅作机制论述的学术佐证，不含任何绕过实现。

---

*下一篇：[验证流程形态学：账号注册的几种"门"](./02-verification-flow-patterns.md)*
