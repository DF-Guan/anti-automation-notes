<div align="center">

# Anti-Automation Notes

**A defense-oriented field guide to anti-automation & verification mechanisms on the modern web.**

**Author:** [DF-Guan](https://github.com/DF-Guan) · **License:** [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

</div>

<p align="center">
  <strong>English</strong> · <a href="./README.md">简体中文</a>
</p>

---

## What Is This?

A **defense- and research-oriented** note repository about the engineering practices behind "human vs. machine differentiation" on the modern web—CAPTCHA forms and principles, common account-verification flow patterns, and how servers combine these measures against registration abuse.

> ⚠️ **Scope**: This repository documents **mechanisms and principles only**; it ships **no runnable signup/CAPTCHA-breaking implementation**. Every key claim is backed by publicly-verified academic references. It is intended for:
>
> - **Product/security engineers**: understand the available verification measures and their trade-offs when designing registration flows
> - **Security researchers**: grasp the engineering details of CAPTCHAs and verification flows as a research starting point
> - **Platform operators**: assess the anti-abuse strength of their own registration flow

---

## Table of Contents

| # | Article | 中文 | English |
|---|---|---|---|
| 1 | A Taxonomy of CAPTCHAs: From Images to Invisible Challenges | [中文](./articles/01-captcha-taxonomy.md) | [EN](./articles/01-captcha-taxonomy.en.md) |
| 2 | A Morphology of Verification Flows: The "Gates" of Account Registration | [中文](./articles/02-verification-flow-patterns.md) | [EN](./articles/02-verification-flow-patterns.en.md) |
| 3 | An Abuse-Defense Matrix for Registration: What a Server Can Do | [中文](./articles/03-abuse-defense-matrix.md) | [EN](./articles/03-abuse-defense-matrix.en.md) |

### Article Abstracts

**No. 1 — A Taxonomy of CAPTCHAs**

> Surveys the evolution of CAPTCHA forms—classic text/image challenges, signal-scoring schemes such as reCAPTCHA v2 and hCaptcha, invisible challenges such as Turnstile, and headless-browser fingerprint checks—analyzing per category the recognition principle, human burden, resistance to automation, and each approach's cost ceiling.
>
> **Keywords**: CAPTCHA; human-computer differentiation; device fingerprinting; invisible challenges; anti-abuse

**No. 2 — A Morphology of Verification Flows**

> Compares post-registration verification forms: no-verification signup, email OTP, magic links, SMS verification, knowledge-based Q&A, and manual approval. Central thesis: the strength of a verification flow is essentially the **acquisition cost of the verification object (email / phone number / human review)**—not the CAPTCHA itself.
>
> **Keywords**: account verification; email OTP; magic link; disposable email; manual approval

**No. 3 — An Abuse-Defense Matrix for Registration**

> Maps the full server-side defense stack—entry layer, form/route layer, identity layer, policy layer, data layer—arguing that "strength comes from combination and depth" and that measures are substitutable, and closes with a legitimate, escalating checklist for assessing a registration flow's strength.
>
> **Keywords**: anti-abuse; layered defense; rate limiting; IP reputation; disposable email; behavioral risk control

---

## Key Concepts at a Glance

```
Anti-automation layer stack:

  L1  Invisible  — invisible challenges: Turnstile / hCaptcha Enterprise / behavioral fingerprints
  L2  Visible    — visible CAPTCHAs: reCAPTCHA v2 / image selection / slider puzzles
  L3  Identity   — account verification: email OTP / magic link / SMS / knowledge Q&A / manual approval
  L4  Policy     — policy layer: email-domain blocking / rate limiting / network policy / IP reputation
```

---

## Repository Structure

```
anti-automation-notes/
├── README.md                          # This file (Simplified Chinese)
├── README.en.md                       # English version
├── LICENSE                            # CC BY-NC 4.0 license text
└── articles/
    ├── 01-captcha-taxonomy.md         # Article 1 (中文)
    ├── 01-captcha-taxonomy.en.md      # Article 1 (English)
    ├── 02-verification-flow-patterns.md       # Article 2 (中文)
    ├── 02-verification-flow-patterns.en.md    # Article 2 (English)
    ├── 03-abuse-defense-matrix.md             # Article 3 (中文)
    └── 03-abuse-defense-matrix.en.md          # Article 3 (English)
```

---

## Writing Principles

- **Mechanism-first**: describe only "what it is, why it exists, and what it costs"—never "how to bypass"
- **Anonymized examples**: no specific operators named; only generic forms described
- **Honest assessment**: every strength/weakness claim comes with reasoning and limitations; no "silver bullet" narrative
- **Traceable**: key claims carry academic references (see each article's References), all verified via public retrieval

---

## Citation Note

- Each article numbers its references independently, cited inline as superscript `[n]`, with a "References / 参考资料" section at the end
- Citation format follows **IEEE reference conventions** (authors, title, conference/journal, volume/pages, DOI)
- Page/volume numbers follow the original publications; references are publicly verified, never fabricated

---

## License

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) (Attribution-NonCommercial). Citation for learning is allowed; use for building automated-abuse tooling is prohibited.

---

<p align="center"><strong>English</strong> · <a href="./README.md">简体中文</a></p>
