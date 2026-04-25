# CLAUDE.md — Behavioral rules

> Two-mode rulebook: Karpathy 4 principles for **writing code**, audit workflow for **reviewing**.
> Full checklist (sections 0–10) lives in [SPEC.md](./SPEC.md). This file is the entry-point + triggers.
>
> **Always respond in the user's language.** This file is in English, but if the user writes in Korean / Japanese / Chinese / etc., translate output on the fly. Item names from SPEC.md may be translated; keep `#` numbers and grade emojis as-is.

---

## Part A — Karpathy 4 Principles (whenever writing code)

### 1. Think Before Coding
- State assumptions explicitly. If uncertain, ask.
- Multiple interpretations? Present them — don't pick silently.
- A simpler approach exists? Say so. Push back when warranted.
- Unclear? Stop. Name what's confusing. Ask.

### 2. Simplicity First
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- 200 lines that could be 50 → rewrite.

### 3. Surgical Changes
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style — even if you'd do it differently.
- Notice unrelated dead code → mention only, don't delete.
- Remove only orphans **your changes** created.

> Test: every changed line must trace directly to the user's request.

### 4. Goal-Driven Execution
Transform tasks into verifiable goals:
| Request | Goal |
|---------|------|
| "Add validation" | "Write tests for invalid inputs → fail → implement → pass" |
| "Fix the bug" | "Write reproducing test → fail → fix → pass" |
| "Refactor X" | "Tests pass → refactor → tests still pass" |

For multi-step tasks, state a brief plan with per-step verify checks.

---

## Part B — Audit Mode Triggers

Activate the audit workflow when the user says any of:
- audit · review · check · verify · assess (any language equivalent)
- pre-launch / production-readiness / 출시 전 / 점검 / 감사 / 리뷰
- skeleton / bootstrap / project setup / 골조 / 셋업
- security review · OWASP · 12-Factor · SOLID · 보안 점검

Workflow:
1. **Get the service profile first.** Ask for missing fields in *one* batch (don't drip-feed).
2. **Match stage → sections.** planning/skeleton → 0 only · building → 0,1,2,6 (🔴🟠) · pre-launch → all · incident → relevant · pre-feature → 1,2,4 + relevant.
3. **Apply grade filter.** Decide if each item applies to *this* service before checking. Skip with one-line reason if not.
4. **Output format**:
   ```
   ✅ [#] item — file:line
   ❌ [#] item — problem + patch
   ⚠️ [#] item — partial
   ⏭️ [#] item — not applicable (one-line reason)
   ```
5. **Re-sort ❌ by risk**: Security (2) > Reliability (1) > Data (4) > Performance (5) > rest. Output as a "fix this first" list.

**Full item descriptions, grade definitions, and the service-profile YAML are in [SPEC.md](./SPEC.md).** Read it when running an audit.

---

## Part C — Conflict resolution

| Situation | Apply |
|-----------|-------|
| User asks for audit/review | Part B — read SPEC.md, follow workflow |
| User asks for code edits/features | Part A — Karpathy 4 |
| Audit found ❌ → providing fix code | Part A applies (surgical, simple) |
| Tempted to fix something unrelated | Mention only — don't touch (rule 3) |

---

## Part D — Language adaptation

- Detect user's language from their last message.
- All AI output (explanations, audit findings, code comments if user comments are in their language) → user's language.
- Code identifiers, file paths, grade emojis (🔴🟠🟡🔵⚪✅❌⚠️⏭️), section/item numbers → keep as-is.
- SPEC.md item names: translate the *display name* in output, but reference the original number (e.g., `❌ 2.10 CORS configuration` → `❌ 2.10 CORS 설정`).
