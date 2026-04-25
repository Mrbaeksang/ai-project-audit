# ai-project-audit

> **Karpathy's 4 coding principles + a 10-section production audit**, packaged as one Claude Code plugin. Tell it your service profile — it tells you what's missing.

[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-orange)](https://docs.anthropic.com/claude-code) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) [![Multilingual](https://img.shields.io/badge/i18n-EN%20%7C%20한국어-green)](README.ko.md)

🇰🇷 **[한국어 README](README.ko.md)**

---

## What it does

Two behaviors injected into Claude Code, in one install:

1. **Karpathy Discipline** — Think before coding · Simplicity first · Surgical changes · Goal-driven execution. Stops drive-by refactors and over-engineering. ([source](https://x.com/karpathy/status/2015883857489522876))
2. **Project Audit** — You give a 10-line service profile (MAU, stage, sensitivity, etc.). It runs a 0–10 section checklist (OWASP / SOLID / ACID / 12-Factor / …), filters by grade (🔴🟠🟡🔵⚪), and outputs `✅ ❌ ⚠️ ⏭️` per item — sorted by risk.

**Multilingual**: skills are written in English, but **respond in your language**. Ask in Korean → get Korean. No config.

---

## Install (30 seconds)

```text
/plugin marketplace add Mrbaeksang/ai-project-audit
/plugin install ai-project-audit@mrbaeksang-marketplace
```

Or single-file (no plugin needed):

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/Mrbaeksang/ai-project-audit/main/CLAUDE.md
```

---

## Use

```text
/audit ./src         # Full audit — asks for service profile, then checks 0–10
/skeleton .          # Bootstrap section 0 only — before any feature code
```

Or just talk: *"audit this for OWASP"*, *"pre-launch review"*, *"set up project skeleton"*, *"check 12-Factor compliance"*.

### Output you get

```
✅ 1.1 Health endpoint — src/health.controller.ts:12
❌ 2.10 CORS — src/main.ts:8 (wildcard `*` with credentials)
   fix: app.enableCors({ origin: ['https://app.example.com'], credentials: true })
⚠️ 5.2 DB index — users.email has no index (login hot path)
⏭️ 8.4 read replica — MAU 50k still fits one DB

Fix this first (by risk):
1. ❌ 2.10 CORS  ← security
2. ❌ 1.4 timeouts ← reliability
...
```

---

## What's inside

| File | Role |
|------|------|
| **[SPEC.md](SPEC.md)** | Full 0–10 checklist (the source of truth AI reads) |
| **[CLAUDE.md](CLAUDE.md)** | Behavioral rules (Karpathy 4 + audit triggers) |
| **[EXAMPLES.md](EXAMPLES.md)** | Concrete usage scenarios |
| `skills/karpathy-discipline/` | Auto-fires on any code edit |
| `skills/project-audit/` | Auto-fires on audit/review keywords |
| `commands/audit.md`, `commands/skeleton.md` | `/audit`, `/skeleton` |

### The 10 audit sections

`0` Skeleton · `1` Reliability · `2` Security (OWASP) · `3` Architecture (SOLID) · `4` Data (ACID) · `5` Performance · `6` Testing · `7` DevOps · `8` Scalability · `9` 12-Factor · `10` Docs

---

## How it knows what applies to *you*

You fill this once per audit (the skill asks for it):

```yaml
service_type: REST API           # or GraphQL / monolith / microservice / CLI / batch
stage: pre-launch                # or planning / building / in-production
expected_mau: 50000
team_size: 3
data_sensitivity: medium         # PII
uptime_target: 99.9%
deployment: k8s
exposure: internet-facing
```

Then every checklist item is graded against your profile — 🟡 Mid-scale items skip if you're solo, 🔵 Large-scale skip below 100k MAU, etc. **You only see what matters for you.**

---

## License

MIT — see [LICENSE](LICENSE)

## Credits

Inspired by [Andrej Karpathy on LLM coding pitfalls](https://x.com/karpathy/status/2015883857489522876) and [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills). Built on [OWASP Top 10](https://owasp.org/www-project-top-ten/) and [12-Factor App](https://12factor.net/).
