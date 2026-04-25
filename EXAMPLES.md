# Examples

Four common scenarios. The skill responds in the language you write in.

---

## 1. New project — bootstrap skeleton

**You:**
```
/skeleton .

service_type: REST API
language_framework: FastAPI
stage: skeleton
expected_mau: 10000
team_size: 2-3
data_sensitivity: medium (PII)
deployment: Docker
exposure: internet-facing
```

**Plugin:**
- Section 0 only.
- Tailors 0.1–0.12 to FastAPI/Docker/PII.
- Generates `README.md`, `.env.example`, `pyproject.toml` (Ruff+Black), `.pre-commit-config.yaml`, directory tree.
- **No feature code.**

---

## 2. Pre-launch full audit

**You:**
```
/audit ./src

service_type: REST API (NestJS + PostgreSQL + Redis)
stage: pre-launch
expected_mau: 50000
team_size: 5
data_sensitivity: medium
uptime_target: 99.9%
deployment: k8s
exposure: internet-facing
```

**Plugin:**
- Sections 0–10, grades 🔴🟠🟡 all.
- Per-item:
  ```
  ✅ 1.1 Health endpoint — src/health.controller.ts:12
  ❌ 2.10 CORS — src/main.ts:8 (wildcard `*` + credentials)
       fix: app.enableCors({ origin: ['https://app.example.com'], credentials: true })
  ⚠️ 5.2 DB index — users.email has no index (login hot path)
  ⏭️ 8.4 read replica — MAU 50k still fits one DB
  ```
- Re-sorts ❌ by risk (Security > Reliability > Data > Performance > rest).

---

## 3. Cross-cutting check before a new feature

**You:**
```
About to add a "payments" feature — using ai-project-audit, what cross-cutting items am I likely to miss?
[design sketch]
```

**Plugin:**
- Pulls items from sections 1 (Reliability), 2 (Security), 4 (Data Integrity) relevant to payments.
- Examples: 4.5 Idempotency (prevent double charges), 2.8 Sensitive data masking (no card numbers in logs), 1.10 Retry+backoff (transient PG failures), 4.1 Atomicity (payment+order in one tx).
- Outputs a checkpoint list **before** code is written, so you can build it in.

---

## 4. Incident — focused diagnostic

**You (in Korean):**
```
지금 "프로덕션에서 같은 주문이 두 번 결제되는 이슈" 발생했어.
ai-project-audit의 관련 섹션만 기준으로 점검하고 재발 방지 코드 줘.
```

**Plugin (responds in Korean):**
- Sections 1 (retry/backoff) + 4 (idempotency, optimistic locking, transactions).
- 4.5 멱등성 키 부재 → idempotency-key 헤더 추가 코드.
- 1.10 retry — jitter 누락 시 추가.
- Karpathy rule 3 applies: surgical patch on the payment module only — adjacent code untouched.

---

## Plugin-free usage

If you're not using the plugin, paste **`SPEC.md`** + the YAML profile, and ask:

```
SPEC.md 읽었어. 위 프로필로 [pre-launch full audit / section X only / before-feature check] 점검해줘.
```

The Karpathy 4 rules are in `CLAUDE.md` — drop that into your project-level CLAUDE.md to apply discipline globally.
