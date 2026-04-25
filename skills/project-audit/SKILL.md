---
name: project-audit
description: Service-profile-driven project audit. Auto-fires when the user requests audit, review, code review, pre-launch check, security audit, OWASP/SOLID/12-Factor compliance, project skeleton/bootstrap/setup, or any equivalent in any language (e.g., 점검, 감사, 리뷰, 출시 전 검토, 보안 점검, 골조, 셋업). Reads the full 0–10 section checklist from SPEC.md, filters items by grade (🔴🟠🟡🔵⚪) against the user's service profile, and outputs results as ✅❌⚠️⏭️ markers re-sorted by risk. Respond in the user's language.
---

# Project Audit

This skill carries no checklist content of its own. The full checklist (service profile YAML, grade definitions, stage-to-section matching, 0–10 section items) is in **`SPEC.md`** at the plugin root. Read it when running an audit.

## Workflow

### 0. Get the service profile first
The user's first audit request usually omits the profile. Ask for missing fields in **one batch** (not drip-fed):

```yaml
service_type: ""         # REST API / GraphQL / monolith / microservice / CLI / batch
language_framework: ""
stage: ""                # planning / skeleton / building / pre-launch / in-production
expected_mau: ""
team_size: ""
data_sensitivity: ""     # low / medium (PII) / high (financial/medical/legal)
uptime_target: ""        # none / 99% / 99.9% / 99.99%
deployment: ""
external_deps: ""
exposure: ""             # internet-facing / VPN-only / local-only
business_model: ""
```

> Without the profile, AI defaults to "everything required" or "everything skippable" — both wrong.

### 1. Match stage to sections

| Stage | Sections | Grade filter |
|-------|----------|--------------|
| planning / skeleton | **0** | all |
| building features | 0, 1, 2, 6 | 🔴 + 🟠 |
| pre-launch | 0–10 (all) | all |
| incident | relevant only | all |
| before adding feature | 1, 2, 4 + relevant | 🔴 + 🟠 |

### 2. Apply grade filter
For each item, **first decide if it applies to this service** (based on profile), then check.
- 🔴 Required (always)
- 🟠 Recommended (MAU 1k+ / team 3+ / internet-facing)
- 🟡 Mid-scale (MAU 10k+ / B2B / 99.9% SLA)
- 🔵 Large-scale (MAU 100k+ / microservices / 99.99%)
- ⚪ Optional (skippable with stated reason)

If skipping, output `⏭️` with a one-line reason.

### 3. Output format

```
✅ [#] item — verified at file:line
❌ [#] item — problem + patch code
⚠️ [#] item — partial / weak point
⏭️ [#] item — not applicable for this service (one-line reason)
```

### 4. Sort ❌ by risk
End the audit with a "fix this first" list:
```
Security (2) > Reliability (1) > Data (4) > Performance (5) > rest
```

## Combining with karpathy-discipline

When this skill provides patch code for ❌ items, the `karpathy-discipline` skill applies automatically:
- Surgical patch — only the failing item, not adjacent code
- No 1-use abstractions
- Match existing style
- Mention any related dead code; don't delete it

## Language

Respond in the user's language. Item names from SPEC.md may be translated for display; keep `#` numbers and grade emojis as-is.
