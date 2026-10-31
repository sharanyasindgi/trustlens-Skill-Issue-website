# TrustLens — Pause. Check. Verify.

TrustLens is a small, single-file web app that helps people spot warning signs in suspicious SMS and email messages. Paste a message, and TrustLens shows a risk level, explains the specific red flags it found, and suggests a safe way to verify the sender.

It is designed for an Indian audience: the interface is in English, and the detection rules include keywords in **English, Hindi (हिन्दी) and Kannada (ಕನ್ನಡ)**, plus code-mixed messages (e.g. Hindi + English) and common Indian banks (SBI, HDFC, ICICI, Canara) and BESCOM.

> **Status:** early demo. Detection is rule-based and will miss new scams and occasionally flag legitimate messages. See [Limitations](#limitations).

---

## Features

- **Risk score (0–100)** with a Low / Medium / High level and a colour-coded meter
- **Plain-language findings** explaining *why* a message looks suspicious
- **Link analysis** that reads URLs as text only. Links are never opened or fetched
- **Multilingual keyword rules** for English, Hindi and Kannada
- **Safe next-step advice** that changes with the risk level
- **Three built-in samples** (scam SMS, Hindi/English code-mixed SMS, harmless email)
- **No build step, no dependencies:** one HTML file with inline CSS and JavaScript
- **Responsive layout** for desktop and mobile, with `prefers-reduced-motion` support
- **Keyboard friendly:** `Ctrl/Cmd + Enter` runs the check; focus states and ARIA live regions are included

---

## Getting started

No installation is needed.

1. Save the code as `index.html`.
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari).

Or serve it locally:

```bash
# from the folder containing index.html
python3 -m http.server 8000
# then visit http://localhost:8000
```

It can also be hosted on any static host (GitHub Pages, Netlify, Cloudflare Pages, S3, etc.).

---

## Usage

1. Choose the **message type** (SMS or Email).
2. Optionally pick the **message language** and enter the **sender** (a name or number).
3. Paste the message into the text box (max 1,500 characters).
4. Click **Check for warning signs** (or press `Ctrl/Cmd + Enter`).
5. Review the risk level, the warning signs, and the suggested safe next step.

Use **Clear message** to reset the form, or click a card under **Try a sample** to load an example and run it immediately.

---

## How it works

The analyzer (`analyze(text, sender)`) lowercases the message and applies a set of rules. Each rule that fires adds points to the score (capped at 100) and adds a finding to the results.

| Signal | Points | What triggers it |
|---|---:|---|
| Suspicious link | 32 | URL shortener (`bit.ly`, `tinyurl`, `t.co`, `goo.gl`, `cutt.ly`, `rb.gy`, `is.gd`), punycode (`xn--`), raw IP address, 3+ dots in the host, a hyphen in the host, or a domain that doesn't match a brand named in the message or sender |
| Unusual link address | 22 | Link ends in a throwaway TLD (`.xyz`, `.top`, `.click`, `.work`, `.live`, `.buzz`, `.icu`) |
| Asks for a secret or credential | 38 | Keywords such as `otp`, `pin`, `cvv`, `password`, `passcode`, `verification code`, and Hindi/Kannada equivalents. Skipped if the message says "do not share" / "never share" (or the Hindi/Kannada equivalents) |
| Creates pressure to act quickly | 18 | Keywords such as `urgent`, `immediately`, `blocked`, `suspended`, `expire`, `तुरंत`, `ತಕ್ಷಣ` |
| Pushes you to take an action | 12 | Words like `verify`, `update`, `confirm`, `click`, `pay`, `claim`, `login` (only if the credential rule didn't already fire) |
| Sender identity is unclear | 10 | Sender field contains `unknown`, `private`, `random`, or a long number |

Link rules are mutually exclusive: at most one link finding is added per message.

### Risk levels

| Score | Level | Advice shown |
|---|---|---|
| 75–100 | **High** | Don't click or share codes; open the official app or call the number on your bank card |
| 60–74 | **Medium** | Pause; verify the sender through a phone number or website you find independently |
| 0–59 | **Low** | No obvious warning signs; verify through official channels if money or personal details are requested |

### Brand/domain matching

`brandDomains` maps a brand keyword to its legitimate domain(s). If the message text or sender mentions a known brand and a link's host isn't that domain (or a subdomain of it), the link is flagged.

```js
const brandDomains = {
  sbi: ['sbi.co.in'],
  'state bank': ['sbi.co.in'],
  hdfc: ['hdfcbank.com'],
  icici: ['icicibank.com'],
  canara: ['canarabank.com'],
  bescom: ['bescom.karnataka.gov.in']
};
```

---

## Privacy

- All analysis runs **in your browser**. The message text is never sent to a server.
- Links in messages are parsed as text and **never opened or requested**.
- The "false alarm" button only shows a confirmation toast; nothing is stored or transmitted.
- **One caveat:** the page loads *DM Sans* and *Manrope* from Google Fonts via a CSS `@import`, so the browser contacts Google to fetch fonts (the message itself is not included). To make the app fully offline/self-contained, self-host the fonts or remove the `@import` line. The CSS falls back to `sans-serif`.

---

## Customising

Everything lives in one file. The most common edits:

| To change… | Edit… |
|---|---|
| Urgency keywords | the `urgent` array |
| Credential/sensitive keywords | the `sensitive` array |
| Known brands and their real domains | the `brandDomains` object |
| Point values or thresholds | the `add(...)` calls and the `level` expression inside `analyze()` |
| Sample messages | the `samples` object and the `.sample` buttons in the HTML |
| Colours | CSS variables in `:root` (`--red`, `--green`, `--amber`, …) |
| Message length limit | `maxlength` on `#message` and the `/ 1,500` label |

Adding a new language is mostly a matter of adding keywords to `urgent` and `sensitive`, and a "do not share" phrase to the regex in the credential rule.

---

## Project structure

```
index.html   # markup, styles and script, all in one file
```

Key script pieces:

- `analyze(text, sender)`: scoring engine; returns `{ score, level, flags, advice, links }`
- `showResult()`: reads the form, runs the analysis, renders the results panel
- `escapeHtml()`: escapes all user-derived text before it is inserted as HTML
- `toast()`: small status notifications

---

## Limitations

This is a demo and should not be treated as a security product.

- **Rule-based only.** New scam wording, spelling tricks, or images/QR codes will be missed. A "Low" result never proves a message is genuine.
- **False positives are possible.** Several keywords are broad (e.g. `link`, `free`, `click`, `today`, `earn`, `share`), so legitimate messages can be flagged. Thresholds are loosely tuned: for instance, the English "Account locked" sample may land in the Medium band rather than High.
- **Matching is substring-based**, so short keywords like `pin` can match inside longer words.
- **Language and message-type selectors are informational.** Language auto-detection is not implemented; the analyzer applies the same rules regardless of the selection.
- **Kannada and Hindi coverage is small** compared with English.
- **No domain reputation lookup.** Links are judged only by their text pattern.
- **The excerpt box shows the message as-is;** it doesn't highlight the matched phrases yet.

---

## Ideas for next steps

- Highlight the exact phrases that triggered each finding in the excerpt
- Actually use the language selector (and add auto-detection)
- Word-boundary matching and weighted keywords to cut false positives
- Self-hosted fonts for a fully offline experience
- More languages (Tamil, Telugu, Malayalam, Bengali, Marathi) and more brands
- Unit tests for `analyze()` with a labelled message set
- Report-to-authority guidance (e.g. India's cybercrime reporting portal and the 1930 helpline)

## Team
Team name: SKILL ISSUES


Sharanya Sindgi

Sanika Somani

Satyam Gusain

Sharan Desai
