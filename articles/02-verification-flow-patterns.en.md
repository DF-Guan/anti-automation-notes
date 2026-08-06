# A Morphology of Verification Flows: The "Gates" of Account Registration

> **No.**: 2 · **Author**: [DF-Guan](https://github.com/DF-Guan) · **Language**: English | [中文](./02-verification-flow-patterns.md)
> **License**: CC BY-NC 4.0 · **Scope**: Defense- and research-oriented mechanism notes; contains no runnable implementation

---

## Abstract

Article 1 dealt with pre-registration machine identification (CAPTCHAs). This article examines post-registration verification flows: no-verification signup, email OTP, magic links, SMS verification, knowledge-based Q&A, and manual approval. The central thesis is that the strength of a verification flow is essentially determined by **the acquisition cost of the verification object (email / phone number / human review)**—not by the CAPTCHA itself. It also discusses the false-success phenomenon of "flow completed but no account created," and proposes the only hard success criterion.

**Keywords**: account verification; email OTP; magic link; disposable email; manual approval

---

## 1. Positioning: There Are More "Gates" After the CAPTCHA

CAPTCHAs prevent "bulk"; verification flows prevent "falsity and loss of control." The post-registration flow answers not "are you a bot?" but:

> Can this account reach a real human? Does the password truly belong to its owner?

These are two separate systems: a CAPTCHA makes a **one-time human/computer judgment**; a verification flow provides **identity anchoring and accountability**.

---

## 2. An Overview of Common Forms

```
  No-verification    OTP-based        Link-based (magic-link)    Manual review
  ─────────────────────────────────────────────────────────────────────────►
  zero friction      medium friction  medium friction            high friction
  no verification    email OTP        click link in email        manual approval
  no traceability    SMS OTP          passwordless login         application essay
```

### 2.1 No-Verification Signup

Fill in the form and the account is created—no verification step.

**Costs/Risks**
- For platforms: accounts can be generated without limit and at no cost → highest risk of spam accounts, inflated metrics, and promotional abuse
- For humans: zero friction, best experience
- Status: common in tools/docs/developer services (password managers, code hosting, note-taking), which compensate with **downstream behavioral risk control**

### 2.2 Email OTP

After registration, a 4–8 digit code is emailed; the user enters it to complete signup.

**Key Mechanism**
- It verifies "**you can read this email**"; the cost is the email address itself—disposable email/domain pools lower the bar
- A common countermeasure: **blocking disposable-email domains** (validated at signup time; hits are rejected). The risk surface has empirical support: Hu, Peng, and Wang's measurement study at IEEE S&P 2019 collected 2.3 million emails from 7 disposable email services (from ~210K sender domains) and found such accounts are easily **hijacked**—anyone can take them over by sending a password-reset link to the publicly readable temporary address<sup>[1]</sup>. Blocking these domains therefore deters both abuse and the legal/trust risk of stolen accounts
- Strength ceiling: depends on "the acquisition cost of the email account," not on the CAPTCHA itself

> Observation: extracting an OTP is an engineering detail—the verification-code input sometimes has no name/placeholder/aria semantics at all, so it can only be located via **combined features** such as `maxlength` (4–8 digits) plus ancestor text ("verification code"). This indirectly shows that "how precisely a program can locate the input field" is itself a hidden adversarial dimension.

> The other side of the coin: domain blocking relies on **blacklists**, which are inherently slow to cover newly registered domains. Alanazi & Alanazi (2024) proposed identifying disposable emails with NLP + domain validation instead of pure blacklists, reaching 97% accuracy on new domains—evidence that pure list-based approaches have a time gap and dynamic feature detection is a more modern upgrade<sup>[2]</sup>.

### 2.3 Magic Link / Click-Link Verification

The email contains a **link**; clicking it confirms within the browser.

**Key Mechanism**
- It verifies "**the browser holding the email session**"—more direct than typing a numeric code
- Links are usually **single-use and short-lived** (tens of minutes to 72 hours)
- Common variants: **set-password link** ("click to complete signup and set your password"), **passwordless login** (no password; click a link each time to enter)

**Costs**
- For platforms: cannot force users to memorize a password; suited to low-risk scenarios
- For automation: numeric codes can be read by programs; link-based flows require **a real click-and-redirect**, a slightly higher bar—but still do not prevent "the email account itself being held by automation." Disposable-email domain pools are finite and theoretically enumerable, which is exactly why domain blocking and blacklists exist<sup>[1][2]</sup>

### 2.4 SMS OTP

A code sent to a phone number; currently one of the mainstream ways to **strongly bind to real identity**.

**Key Mechanism**
- In most regions phone numbers are **real-name / quasi-real-name** resources, far costlier to obtain than email
- Countermeasures faced: SMS-activation platforms, virtual number segments, overseas card pools—operators must continuously update number-segment blacklists
- Compliance: under GDPR/CCPA/PIPL, collecting phone numbers carries greater data-responsibility obligations

**Costs**
- For humans: the heaviest (owning a phone + waiting for SMS + possible fees); international users may face delayed or missing international SMS
- For platforms: high SMS cost, and the SMS channel itself is an attack surface (API abuse/SMS flooding)

### 2.5 Knowledge-Based Q&A

Answer a question "only a real human would know" during signup; common in anti-spam mechanisms of legacy forum systems.

**Two Typical Forms**
- **Static knowledge questions**: e.g., "Is this community's service paid?"—the answer is fixed and publicly discoverable. For programs this is a "lookup question"; strength depends on whether the question is obscure/dynamic
- **Dynamic computation questions**: e.g., "Compute the output of command xxx"—requires executing an **external command** for the answer. These are a hard barrier to automation (requiring real external code execution), but also a heavy burden on humans and raise clear security-boundary concerns

**Costs**
- For humans: generally disliked, poor compatibility
- For platforms: small question pools get enumerated; large pools raise maintenance costs
- Status: concentrated in self-hosted forums/communities as a low-cost anti-spam measure

### 2.6 Manual Approval / Application Review

After submitting a registration request, a moderator/operator **manually approves**; the account is only created upon approval.

**Key Mechanism**
- Applications usually require an **essay on purpose/self-introduction**—the hardest part for automation to forge (generating a "plausible long-text reason" is far harder than filling forms)
- Common in small communities, invite-only services, and research platforms (federated instances, closed communities)

**Costs**
- For humans: high friction, uncertain wait time
- For platforms: high labor cost, poor scalability—but in exchange, **community quality and account trustworthiness**, usually paired with manual auditing

---

## 3. An Easily Misjudged Phenomenon: The Flow Looks Complete, But No Account Exists

The most dangerous **false positive** when studying registration flows: the flow appears to complete (fill form → receive code → submit), yet the account is never created. Common causes:

- **Code-login single sign-on**: the site uses "email + code" to *log in*, not register; after entering the code it returns to login/homepage—**there was never a signup action**. The page shape is almost indistinguishable from registration by UI alone
- **Silent blocking**: the server silently drops the request (no error, no redirect); the form stays put and no email ever arrives
- **Fake success page**: redirected to a "welcome / almost done" page, but the account does not exist in the records

> Defensive insight: **"completing the flow" is not "account created."** When evaluating a registration flow, the only hard criterion is—**whether these credentials can actually log in**, and **whether the mailbox receives a server-issued email carrying an account-ownership identifier**. Judging success from page copy is the most common pitfall in automated testing.

---

## 4. Comparison Table

| Form | Human burden | Platform cost | Anti-bulk | Traceability | Typical scenario |
|---|---|---|---|---|---|
| No-verification | very low | low | very low | none | tools/docs services |
| Email OTP | low | low | medium (via domain blocking) | email | mainstream signup |
| Magic link | low–medium | low | medium | email | passwordless/low-risk |
| SMS OTP | high | medium–high | high | phone number | finance/real-name |
| Knowledge Q&A | medium | low–medium | medium–high | none | communities/forums |
| Manual approval | high | high | very high | strong | closed communities/research |

---

## 5. Conclusion

- **The strength of a verification flow is essentially the acquisition cost of its verification object (email/phone/human)**—not the CAPTCHA itself
- Modern strong platforms generally **combine**: invisible CAPTCHA (against bots) + email/SMS verification (against falsity) + downstream risk control (against abuse), layered progressively
- For testers and researchers: whether a registration flow is "truly complete" should always be judged solely by "credentials can log in"

---

## References

1. H. Hu, P. Peng, and G. Wang, "Characterizing pixel tracking through the lens of disposable email services," in *IEEE Symposium on Security and Privacy (S&P 2019)*, IEEE, 2019, pp. 365–379. DOI: [10.1109/SP.2019.00033](https://doi.org/10.1109/SP.2019.00033)
2. R. Alanazi and S. Alanazi, "A hybrid NLP and domain validation technique for disposable email detection," *Alexandria Engineering Journal*, vol. 102, pp. 200–210, 2024. DOI: [10.1016/j.aej.2024.05.068](https://doi.org/10.1016/j.aej.2024.05.068)

> Note: All references above were verified through public retrieval. Citation format follows IEEE reference conventions; page/volume numbers follow the original publications. This article only provides scholarly support for mechanism discussion and contains no bypass implementations.

---

*Next: [An Abuse-Defense Matrix for Registration: What a Server Can Do](./03-abuse-defense-matrix.en.md)*
