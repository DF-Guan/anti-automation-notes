# A Taxonomy of CAPTCHAs: From Images to Invisible Challenges

> **No.**: 1 · **Author**: [DF-Guan](https://github.com/DF-Guan) · **Language**: English | [中文](./01-captcha-taxonomy.md)
> **License**: CC BY-NC 4.0 · **Scope**: Defense- and research-oriented mechanism notes; contains no runnable implementation

---

## Abstract

CAPTCHAs (Completely Automated Public Turing tests to tell Computers and Humans Apart) are the standard infrastructure for distinguishing humans from programs on the modern web. From a defensive perspective, this article surveys the evolution of CAPTCHA forms: classic text/image challenges, signal-scoring schemes such as reCAPTCHA v2 and hCaptcha, invisible challenges such as Turnstile, and headless-browser fingerprint checks. Drawing on both the academic literature and engineering practice, it analyzes, per category, the recognition principle, the burden on humans, the resistance to automation, and each approach's ceiling of cost.

**Keywords**: CAPTCHA; human-computer differentiation; device fingerprinting; invisible challenges; anti-abuse

---

## 1. Introduction: Why CAPTCHAs Exist

CAPTCHAs exist for one purpose: **to automatically tell humans from programs without human review**. Registration, login, commenting, and voting—once drivable in bulk by programs—produce spam, fake accounts, and inflated metrics.

The notion of an "automated Turing test" was formalized by von Ahn et al. in 2003<sup>[1]</sup>: a sound CAPTCHA must satisfy—a computer program can generate and grade the test automatically; virtually all humans pass it; and state-of-the-art computer programs at the time essentially cannot. In 2004 they elaborated the definition in *Communications of the ACM*<sup>[2]</sup>.

Every CAPTCHA is ultimately a **game of costs**:

- For humans: the verification cost must be **low enough**, or users churn
- For programs: the recognition/bypass cost must be **high enough**, or the measure is meaningless
- For operators: integration cost, collateral false positives, and compliance risk must stay manageable

This game drives the entire evolution of CAPTCHAs—**from "posing problems to people" toward "checking programs."**

---

## 2. A Spectrum of Types

```
               Posing to "humans" ────────────────► Checking "programs"
   Text/image   Image-selection  Slider   Behavior  Device fingerprint  Invisible
   high friction / high cost       ──►              low friction / low cost
```

### 2.1 Classic Text/Image CAPTCHAs

The earliest form: distorted characters plus noise lines that the user must type; later extended to **image classification** (e.g., "select all street signs / crosswalks / traffic lights").

**Characteristics**
- A standalone image resource or input field, often named with `captcha`
- Requires one genuine visual interaction

**Costs**
- For humans: heavy interference, particularly poor for accessibility
- For operators: OCR/deep-learning models crack distorted text at high rates—"labor-intensive yet unreliable." Historically, OCR could not read about 20% of words in older, faded print; von Ahn et al. repackaged "human recognition" as crowdsourcing—people digitize old books while solving CAPTCHAs, i.e., reCAPTCHA<sup>[3]</sup>
- Status: largely retired from mainstream commercial sites, lingering in legacy forum systems (e.g., self-hosted questions on old BBS platforms)

### 2.2 reCAPTCHA v2 ("I'm not a robot" checkbox)

A single checkbox usually passes. The principle: **collect a large set of browser interaction signals around the click** (mouse trajectory, click history, cookies, page behavior) and score them with a backend model.

**Characteristics**
- A visible checkbox plus brand watermark
- Upgrades to an **image challenge** ("select all squares containing traffic lights") when signals are insufficient

**Costs**
- For humans: very low (usually one click)
- For programs: bypassable by pre-rendering simulated interaction signals—**it defends against "mechanical operation," not against "simulated humans"**
- Limitations: depends on vendor model quality; a constant arms race each year. Bursztein et al.'s CHI 2014 measurements showed human accuracy reaching 95.3% for a more usable CAPTCHA designed for Google, and they explicitly argued that **CAPTCHAs should not be used in isolation**, but as one component of a broader anti-abuse system<sup>[4]</sup>

### 2.3 hCaptcha

Same family as reCAPTCHA, differing in business model and form: it sells "human-labeled data" as its core product, serves a higher share of image-selection tasks, and offers deeply customizable styling (including an invisible enterprise mode).

**Characteristics**
- Same checkbox/image-challenge forms
- Some sites load hCaptcha **only on submit**—i.e., "appears only after the whole form is filled and submit is clicked," noticeably increasing the frustration of automation

**Costs**
- For humans: experience degrades when image tasks are frequent
- For programs: same "signal-scoring" family as reCAPTCHA; strength depends on the vendor model

### 2.4 Turnstile (Cloudflare)

The representative of "invisible challenges": **no visible control on the page**; the backend evaluates request trustworthiness and issues a token directly when trusted, escalating to an interactive challenge only when untrusted.

**Characteristics**
- Often appears as a hidden field `cf-turnstile-response` with no visible element
- Relies on **egress IP reputation**: if an IP has been flagged (e.g., triggered many challenges), the token may stay empty and never materialize
- Tightly bound to Cloudflare's full protection stack (WAF, proxy, CDN)

**Costs**
- For humans: near zero
- For programs: **IP reputation becomes the decisive variable**—a clean egress often passes directly; consequently it is unfriendly to shared/egress proxy environments (causing broad collateral false positives)

> Observation: Turnstile's "invisibility" comes from **shifting the cost to the network and reputation layers**. It does not outwit programs; it fights "egress environments." That is the most essential difference from the reCAPTCHA family.

### 2.5 Headless & Fingerprint Checks

Strictly not CAPTCHAs, but "JS challenges + fingerprint collection":

- The page first runs a script (e.g., computing a math task, rendering canvas) while collecting `navigator`, `screen`, fonts, and Canvas fingerprints
- Missing or inconsistent features (headless browsers, disabled JS, abnormal WebGL rendering) → judged suspicious
- Common forms: `Performing security verification…` loading animations, blank pages, `NS_ERROR_NET_RESET`

**Costs**
- For humans: transparent and invisible
- For programs: targets known signatures of "automation browsers"; requires continuous adaptation to detection-rule updates
- Mathematical basis of fingerprinting: Eckersley's 2010 study measured roughly 18.1 bits of entropy in browser fingerprints, with about 94.2% of browsers unique when Flash/Java are present<sup>[5]</sup>—device fingerprints do have discriminative power, but that very uniqueness implies high false-positive and privacy costs for users of privacy-preserving browsers, enterprise networks, and shared IPs

---

## 3. A Practical View: Recognizing a CAPTCHA Type from Page Structure

For research/ops, quickly classifying a CAPTCHA from page structure is useful. Below are **purely static-observation signals** (involving no bypass):

| Type | Static signals |
|---|---|
| Turnstile | Hidden field `cf-turnstile-response`; URL contains `challenges.cloudflare.com/turnstile` |
| reCAPTCHA | `g-recaptcha-response`; `.g-recaptcha` container; frame contains `recaptcha` |
| hCaptcha | Frame contains `hcaptcha`; `[class*='hcaptcha']` |
| Image CAPTCHA | Input name/class contains `captcha` + an image resource appears |
| Fingerprint challenge | No form, only a loading animation/blank page; network-level resets |

---

## 4. Summary of Evolutionary Patterns

1. **From "posing questions" to "checking"**: making humans solve puzzles is too costly for UX, so commercial sites have fully moved toward behavioral/environmental scoring of programs. The hard data: Bursztein et al. argued that once automated success exceeds about 1%, a CAPTCHA is practically insecure<sup>[6]</sup>; their WOOT 2014 study further showed classic text CAPTCHAs are nearing their end—without any parameter tuning, a generic machine-learning method cracked Yahoo at 5.33%, ReCaptcha at 22.67%, eBay at 51.39%, and CNN at 51.09%, all far above the 1% threshold<sup>[7]</sup>. Audio CAPTCHAs are similarly fragile: a dedicated system broke Microsoft's at about 48.9% and Yahoo's at about 45.5%, often outperforming humans<sup>[8]</sup>
2. **Reputation is the new battlefield**: IP reputation, device reputation, and account-history reputation are replacing "a single puzzle."
3. **Multi-factor combination > single point**: strong sites layer "form validation + invisible challenge + follow-up verification flow" (see article 2)
4. **There is no silver bullet**: each type has a ceiling of cost—perceptive types hurt UX, invisible types depend on reputation, fingerprint types depend on signature databases. That is why they coexist today

## 5. Further Thoughts

- **Accessibility and compliance**: purely visual CAPTCHAs are a barrier to disabled users; WCAG and GDPR/CCPA pressure vendors to provide accessible and privacy-friendly alternatives—a key reason invisible schemes like Turnstile gain adoption.
- **Cost asymmetry**: CAPTCHA vendors "pose puzzles for all of humanity," and defenders are always chasing. Understanding this explains why "CAPTCHA + verification flow + policy" must be combined.

---

## References

1. L. von Ahn, M. Blum, N. J. Hopper, and J. Langford, "CAPTCHA: Using hard AI problems for security," in *Advances in Cryptology — EUROCRYPT 2003*, LNCS 2656, Springer, 2003, pp. 294–311. DOI: [10.1007/3-540-39200-9_18](https://doi.org/10.1007/3-540-39200-9_18)
2. L. von Ahn, M. Blum, and J. Langford, "Telling humans and computers apart automatically," *Communications of the ACM*, vol. 47, no. 2, pp. 56–60, 2004. DOI: [10.1145/966389.966390](https://doi.org/10.1145/966389.966390)
3. L. von Ahn, B. Maurer, C. McMillen, D. Abraham, and M. Blum, "reCAPTCHA: Human-based character recognition via web security measures," *Science*, vol. 322, no. 5898, pp. 1465–1468, 2008. DOI: [10.1126/science.1160379](https://doi.org/10.1126/science.1160379)
4. E. Bursztein, A. Moscicki, C. Fabry, S. Bethard, D. Jurafsky, and J. C. Mitchell, "Easy does it: More usable CAPTCHAs," in *Proceedings of the SIGCHI Conference on Human Factors in Computing Systems (CHI 2014)*, ACM, 2014, pp. 2637–2646. DOI: [10.1145/2556288.2557322](https://doi.org/10.1145/2556288.2557322)
5. P. Eckersley, "How unique is your web browser?," in *Privacy Enhancing Technologies (PETS 2010)*, LNCS 6205, Springer, 2010, pp. 1–18. DOI: [10.1007/978-3-642-14527-8_1](https://doi.org/10.1007/978-3-642-14527-8_1)
6. E. Bursztein, M. Martin, and J. C. Mitchell, "Text-based CAPTCHA strengths and weaknesses," in *Proceedings of the 18th ACM Conference on Computer and Communications Security (CCS 2011)*, ACM, 2011, pp. 125–138. DOI: [10.1145/2046707.2046724](https://doi.org/10.1145/2046707.2046724)
7. E. Bursztein, J. Aigrain, A. Moscicki, and J. C. Mitchell, "The end is nigh: Generic solving of text-based CAPTCHAs," in *8th USENIX Workshop on Offensive Technologies (WOOT 2014)*, USENIX, 2014.
8. E. Bursztein, R. Beauxis, H. Paskov, D. Perito, C. Fabry, and J. Mitchell, "The failure of noise-based non-continuous audio CAPTCHAs," in *IEEE Symposium on Security and Privacy (S&P 2011)*, IEEE, 2011, pp. 19–31. DOI: [10.1109/SP.2011.14](https://doi.org/10.1109/SP.2011.14)

> Note: All references above were verified through public retrieval. Citation format follows IEEE reference conventions; page/volume numbers follow the original publications. This article only provides scholarly support for mechanism discussion and contains no bypass implementations.

---

*Next: [A Morphology of Verification Flows: The "Gates" of Account Registration](./02-verification-flow-patterns.en.md)*
