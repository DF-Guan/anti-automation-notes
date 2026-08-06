# CAPTCHA 类型学：从图片到无感挑战

> 防御视角笔记 · 第 1 篇 · **作者：[DF-Guan](https://github.com/DF-Guan)**
> 主题：现代 CAPTCHA 的形态演进与各自的设计权衡

---

## 1. 为什么需要 CAPTCHA

CAPTCHA 的存在意义只有一个：**在无需人工审计的前提下，区分"真人"与"程序"**。注册、登录、评论、投票等场景一旦可以被程序批量驱动，就会出现垃圾内容、虚假账号、刷量等问题。「自动图灵测试」这一概念由 von Ahn 等人在 2003 年正式提出，其定义要求：计算机程序能自动生成并评判测试，且几乎所有人类都能通过、而当时的计算机程序几乎无法通过<sup>[1]</sup>；2004 年他们又在《Communications of the ACM》上给出了更完整的阐述<sup>[2]</sup>。

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
- 对运营方：OCR/深度学习模型对扭曲文字破解率高，属于"劳动密集型但不保险"的手段。早在 2008 年，von Ahn 等人的 reCAPTCHA 论文就已说明：当时的 OCR 对扫描文字的错误率约为 20%，于是他们反过来把「人脑识别」打包成众包——人验证的同时顺便完成数字化任务<sup>[3]</sup>
- 现状：已基本退出主流商业站点，多残留在老旧论坛系统（如一些传统 BBS 自建题）中

### 2.2 reCAPTCHA v2（"I'm not a robot" 复选框）

点一下复选框，多数情况下即通过。原理是**在点击前后采集大量浏览器交互信号**（鼠标轨迹、点击历史、cookies、页面行为），交给后端模型打分。

**特征**
- 可见的复选框 + 品牌水印
- 信号不足时会升级出**图片选择题**（"选中所有包含红绿灯的格子"）

**代价**
- 对真人：很低（多数情况一下点击）
- 对程序：可通过大量预渲染模拟交互信号绕过——**它防的是"机械操作"，防不住"模拟真人"**
- 局限：依赖服务端模型质量，且每年都被攻防拉锯。Bursztein 等人在 CHI 2014 的研究曾系统测过各类验证码的真实通过率（人类可到 95% 以上），并明确提出「验证码不应被孤立使用，而应与其他反滥用机制组合」——这几乎是后来所有厂商的设计共识<sup>[4]</sup>

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
- 指纹的数学基础：Eckersley 在 2010 年的研究用 18.1 bits 熵描述浏览器指纹的信息量——在带 Flash/Java 的环境中，约 94.2% 的浏览器指纹是唯一的；这意味着「设备指纹」确实能撑起区分度，但同时也带来误伤与隐私问题<sup>[5]</sup>

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

1. **从"出题"到"检查"**：让真人做题的体验成本太高，商业站点全面转向"对程序做行为/环境评分"。这背后有硬数据支撑：Bursztein 等人在 USENIX WOOT 2014 证明了「基于文字的传统验证码已到尽头」——通用机器学习方案在所有实测验证码上都超过了 1% 的不安全阈值（Yahoo 5.33%、eBay 3.7%、ReCaptcha 33.34% 等），文章标题直接叫《The End is Nigh》<sup>[6]</sup>；同年另一篇对音频验证码的系统研究也显示其成功率可达 45%-49%<sup>[7]</sup>
2. **信誉成为新战场**：IP 信誉、设备信誉、账号历史信誉，正在替代"一道题"的作用
3. **多因子组合 > 单点**：强站点往往"表单校验 + 无感挑战 + 后续验证流程"层层叠加（见第 2 篇）
4. **没有银弹**：每一类都有各自的成本天花板——感知型伤体验、无感型依赖信誉、指纹型依赖特征库——这就是它们至今共存的原因

## 5. 延伸思考

- **可访问性与合规**：纯视觉 CAPTCHA 对残障用户是障碍；WCAG 与 GDPR/CCPA 都在倒逼厂商提供无障碍与隐私替代方案——这也是 Turnstile 这类"无感方案"被更多站点接纳的原因之一。
- **成本不对称**：验证码厂商是"为全人类发题"，防御方永远在追赶；理解这一点，才会明白为什么"验证码+验证流程+策略"必须组合使用。

---

## 参考资料

1. von Ahn, L., Blum, M., Hopper, N. J., & Langford, J. (2003). *CAPTCHA: Using Hard AI Problems for Security.* In: EUROCRYPT 2003, LNCS 2656, pp. 294–311. DOI: [10.1007/3-540-39200-9_18](https://doi.org/10.1007/3-540-39200-9_18)
2. von Ahn, L., Blum, M., & Langford, J. (2004). *Telling Humans and Computers Apart Automatically.* Communications of the ACM, 47(2), 56–60. DOI: [10.1145/966389.966390](https://doi.org/10.1145/966389.966390)
3. von Ahn, L., Maurer, B., McMillen, C., Abraham, D., & Blum, M. (2008). *reCAPTCHA: Human-Based Character Recognition via Web Security Measures.* Science, 322(5898), 1465–1468. DOI: [10.1126/science.1160379](https://doi.org/10.1126/science.1160379)
4. Bursztein, E., Moscicki, A., Fabry, C., Bethard, S., Jurafsky, D., & Mitchell, J. C. (2014). *Easy Does It: More Usable CAPTCHAs.* In: Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI 2014). ACM.
5. Eckersley, P. (2010). *How Unique Is Your Web Browser?* In: Privacy Enhancing Technologies (PETS 2010), LNCS 6205, pp. 1–18. DOI: [10.1007/978-3-642-14527-8_1](https://doi.org/10.1007/978-3-642-14527-8_1)
6. Bursztein, E., et al. (2014). *The End is Nigh: Generic Solving of Text-based CAPTCHAs.* In: 8th USENIX Workshop on Offensive Technologies (WOOT 2014). USENIX.
7. Bursztein, E., Beauxis, R., Paskov, H., Perito, D., Fabry, C., & Mitchell, J. (2011). *The Failure of Noise-Based Non-continuous Audio CAPTCHAs.* In: IEEE Symposium on Security and Privacy (S&P 2011), pp. 19–31. DOI: [10.1109/SP.2011.14](https://doi.org/10.1109/SP.2011.14)

> 说明：以上文献均来自公开检索，仅作机制论述的佐证；标注形式遵循常见引用规范，页码/卷期以原刊为准。

---

*下一篇：[验证流程形态学：账号注册的几种"门"](./02-verification-flow-patterns.md)*
