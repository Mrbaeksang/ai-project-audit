# SPEC — AI Project Audit Checklist (0–10)

> Full source-of-truth for the `project-audit` skill. AI loads this when a user requests an audit.
>
> **Respond in the user's language.** This file is in English, but if the user writes in Korean, Japanese, etc., translate findings on the fly.

---

## ① Service Profile (fill this first)

> Without this, AI mistakenly applies "everything required" or "everything skippable".

```yaml
service_type: ""        # REST API / GraphQL / monolith / microservice / internal / CLI / batch
language_framework: ""  # FastAPI / NestJS / Django / Express / Spring / etc.
stage: ""               # planning / skeleton / building features / pre-launch / in-production
expected_mau: ""        # 100 / 10,000 / 1M
team_size: ""           # solo / 2-3 / 5+ / 10+
data_sensitivity: ""    # low (public) / medium (PII) / high (financial / medical / legal)
uptime_target: ""       # none / 99% / 99.9% / 99.99%
deployment: ""          # local / single-server / Docker / k8s / serverless
external_deps: ""       # DBs / Redis / S3 / external APIs
exposure: ""            # internet-facing / VPN-only / local-only
business_model: ""      # B2C / B2B / internal / side-project
```

## ② Grade Levels

> Each item carries a grade — "must have if this condition holds". AI must first decide if the item applies to this service based on the profile, then check. If skipped (⏭️), state a one-line reason.

| Grade | Meaning |
|-------|---------|
| 🔴 **Required** | All projects, regardless of scale |
| 🟠 **Recommended** | MAU 1k+ / team 3+ / internet-facing |
| 🟡 **Mid-scale** | MAU 10k+ / B2B / 99.9% SLA |
| 🔵 **Large-scale** | MAU 100k+ / microservices / 99.99% SLA |
| ⚪ **Optional** | Skippable with a stated reason |

## ③ When to Apply (by stage)

| Stage | Sections to apply | Goal |
|-------|-------------------|------|
| **[1] Pre-start** | Section **0** + profile | Lock in skeleton/rules before code |
| **[2] After 1–2 features** | 0, 1, 2, 6 (🔴🟠 only) | Minimum to not miss before scaling |
| **[3] Pre-launch** | All sections | Production-readiness sweep |
| **[4] Incident** | Relevant sections only | Focused diagnostic |
| **[5] Before adding a feature** | 1, 2, 4 + relevant | Cross-cutting concerns check |

## ④ Result Format

For each checklist item:
```
✅ [#] item — verified at (file:line)
❌ [#] item — problem + patch code
⚠️ [#] item — partial / weak
⏭️ [#] item — not applicable for this service (one-line reason)
```

After listing, **re-sort ❌ items by risk: Security (2) > Reliability (1) > Data (4) > Performance (5) > rest** as a "fix this first" list.

---

## 0. Project Skeleton (lock in before any feature code)

> **When**: immediately after starting, **before a single line of feature code**. Skipping here creates "rewrite the directory structure" / "rename the world" / "polluted git history" — non-recoverable tech debt.

### Skeleton checklist

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 0.1 | **Directory structure** | Pick one — layered (controller/service/repo) · feature-based · domain-based — and document in README | 🔴 | — |
| 0.2 | **Dependency direction rules** | Codify which layer can import which (e.g., controller → service → repo, no reverse) | 🟠 | < 50-line scripts |
| 0.3 | **Naming conventions** | Files (kebab/PascalCase) · vars (camelCase) · constants (UPPER_SNAKE) · endpoints (plural lowercase) — unify | 🔴 | — |
| 0.4 | **Git branching strategy** | main/develop/feature OR trunk-based; protection rules (no force-push, etc.) | 🔴 | solo/local-only |
| 0.5 | **Commit message convention** | Conventional Commits (`feat:`, `fix:`, `refactor:`) or team rules; one language | 🟠 | — |
| 0.6 | **Linter / formatter / pre-commit** | ESLint+Prettier / Ruff+Black; husky/lefthook | 🔴 | — |
| 0.7 | **README setup guide** | Anyone can run locally in 10 min; commands ≤ 5 lines | 🔴 | — |
| 0.8 | **.env.example** | Required env vars listed; placeholders for secrets; matches docker-compose | 🔴 | no env vars |
| 0.9 | **Error code system** | Custom error class + enum (`AUTH_INVALID_TOKEN`); shared frontend format | 🟠 | trivial scripts |
| 0.10 | **API contract** | OpenAPI/Zod/Pydantic for FE-BE type sharing; versioning policy (v1/v2) | 🟠 | single client |
| 0.11 | **One-command local run** | `pnpm dev` / `make dev` boots infra + app | 🟠 | — |
| 0.12 | **Dependency addition criteria** | Review checklist for new packages (maintenance, license, size, alternatives) | 🟡 | personal projects |

### Directory structure decision guide (0.1 detail)

| Structure | Best for | Example |
|-----------|----------|---------|
| **Layered** | Small-mid monolith, simple CRUD | `src/{controllers,services,repositories}` |
| **Feature-based** | Clear feature boundaries, feature teams | `src/features/{auth,billing,search}` |
| **Domain-based (DDD)** | Complex domain, enterprise | `src/domains/{order,inventory}/*/{api,domain,infrastructure}` |

**Principle**: pick a structure you won't regret at 200+ files. Changing mid-flight is **hell**.

### Dependency direction example (0.2 detail)

```
allowed:    controller ─→ service ─→ repository ─→ model
forbidden:  repository ─→ service     (reverse)
            service ─→ controller     (upward violation)
            feature A ─→ feature B    (horizontal coupling — extract to shared)
```

Enforce via lint rules (`dependency-cruiser`, `nx` boundaries). **Automated rules beat README docs.**

---

## 1. Operational Reliability

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 1.1 | **Health endpoint** | `/health` — DB/Redis included, liveness/readiness split | 🔴 | CLI / local batch |
| 1.2 | **Env var startup validation** | App fails fast on missing required ENV | 🔴 | — |
| 1.3 | **Global error handler** | All unhandled exceptions → structured log → proper HTTP response | 🔴 | — |
| 1.4 | **External dep timeouts** | DB, HTTP client, Redis — explicit timeouts | 🔴 | no external deps |
| 1.5 | **Graceful shutdown** | SIGTERM → drain → close DB pool | 🔴 | CLI / batch |
| 1.6 | **Structured logging** | JSON, level, timestamp, service name | 🔴 | local dev only |
| 1.7 | **Correlation ID** | request_id propagated across logs/responses | 🟠 | single function / CLI |
| 1.8 | **OpenTelemetry** | trace/span/metric via OTLP | 🟠 | single service / internal |
| 1.9 | **/metrics endpoint** | Prometheus format — request count, latency, error rate | 🟠 | no k8s/cloud |
| 1.10 | **Retry + exponential backoff** | Transient-failure auto-recovery, with jitter | 🟠 | no external deps |
| 1.11 | **Circuit breaker** | Prevent cascade failure from external API outage | 🟡 | ≤ 1 external dep |
| 1.12 | **Dead letter queue** | Preserve and replay failed async jobs | 🟡 | no queue |
| 1.13 | **Alerting** | Threshold-based alerts (Slack/PagerDuty) | 🟠 | side project |
| 1.14 | **Rate limiting** | Per-endpoint request limits | 🟠 | VPN-only |
| 1.15 | **Request size limit** | body / file upload caps | 🔴 | — |

---

## 2. Security (OWASP Top 10)

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 2.1 | **No hardcoded secrets** | API keys, DB passwords in `.env` / secret manager only | 🔴 | — |
| 2.2 | **Input validation** | Type/format/length checks at every boundary | 🔴 | — |
| 2.3 | **SQL injection prevention** | Parameterized queries / ORM only — no raw queries | 🔴 | no DB |
| 2.4 | **Authentication (AuthN)** | Session/JWT/OAuth with expiration | 🔴 | fully public API |
| 2.5 | **Authorization (AuthZ)** | Resource ownership checks, RBAC | 🔴 | single user |
| 2.6 | **Enforce HTTPS** | HTTP → HTTPS redirect, HSTS header | 🔴 | VPN-only |
| 2.7 | **No info leak in errors** | No stack traces / DB errors in responses | 🔴 | — |
| 2.8 | **Sensitive data masking** | No PII / card numbers / passwords in plaintext logs | 🔴 | no sensitive data |
| 2.9 | **Password hashing** | bcrypt / argon2 — never plaintext | 🔴 | no own auth |
| 2.10 | **CORS configuration** | Explicit origins, no wildcard `*` with credentials | 🟠 | no browser client |
| 2.11 | **XSS prevention** | Output escaping, CSP header | 🟠 | no HTML rendering |
| 2.12 | **CSRF prevention** | CSRF token / SameSite cookie | 🟠 | no cookie auth |
| 2.13 | **Brute-force prevention** | Login attempt limits, account lockout | 🟠 | no auth |
| 2.14 | **Dep vulnerability scan** | `npm audit` / `pip-audit` / Snyk in CI | 🟠 | side project |
| 2.15 | **Least privilege** | DB user / IAM role / service account — minimum perms | 🟠 | — |
| 2.16 | **File upload validation** | Extension / MIME / size checks; hide storage path | 🔴 | no uploads |

---

## 3. Code Architecture

### SOLID

| # | Principle | Description | Grade | Skippable when |
|---|-----------|-------------|-------|----------------|
| 3.1 | **SRP** | One reason to change per class/function | 🟠 | < 50-line scripts |
| 3.2 | **OCP** | Open to extension, closed to modification | 🟡 | — |
| 3.3 | **LSP** | Subclasses substitutable for base | 🟠 | no inheritance |
| 3.4 | **ISP** | Don't depend on unused methods | 🟠 | — |
| 3.5 | **DIP** | Depend on abstractions, use DI | 🟠 | trivial scripts |

### General principles

| # | Principle | Description | Grade | Skippable when |
|---|-----------|-------------|-------|----------------|
| 3.6 | **DRY** | Same logic in 2+ places → abstract | 🟠 | — |
| 3.7 | **YAGNI** | No features / abstractions without current need | 🔴 | — |
| 3.8 | **KISS** | Functions ≤ 50 lines, nesting ≤ 4 levels | 🟠 | — |
| 3.9 | **Immutability** | Return new objects, don't mutate | 🟠 | hot performance paths |
| 3.10 | **Consistent error handling** | No silent swallowing | 🔴 | — |
| 3.11 | **Clear naming** | Names express intent, minimal abbrev | 🔴 | — |
| 3.12 | **Function size** | ≤ 50 lines / file ≤ 400 lines | 🟠 | — |
| 3.13 | **No circular deps** | Module A → B → A forbidden | 🟠 | — |
| 3.14 | **Null/Optional explicitness** | Type-level nullability, no missed checks | 🟠 | dynamic-typed langs |

---

## 4. Data Integrity

### ACID

| # | Property | Description | Grade | Skippable when |
|---|----------|-------------|-------|----------------|
| 4.1 | **Atomicity** | Multi-step DB writes in one transaction | 🔴 | single-write ops |
| 4.2 | **Consistency** | Constraints always hold post-tx | 🔴 | — |
| 4.3 | **Isolation** | Concurrent tx don't interfere; isolation level set | 🟠 | single-request |
| 4.4 | **Durability** | Committed data survives crashes | 🔴 | cache / temp data |

### General data principles

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 4.5 | **Idempotency** | Same request N times → same result | 🟠 | read-only API |
| 4.6 | **Optimistic locking** | Detect concurrent edits (version / updated_at) | 🟡 | no concurrent edits |
| 4.7 | **Soft vs hard delete** | Stated strategy, recoverable from accidents | 🟠 | — |
| 4.8 | **Migration strategy** | Reversible, zero-downtime | 🟠 | early dev |
| 4.9 | **Backup strategy** | Frequency, retention, recovery procedure documented | 🟠 | data loss OK |
| 4.10 | **Boundary input validation** | Domain rules checked before persistence | 🔴 | — |

---

## 5. Performance

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 5.1 | **N+1 query prevention** | ORM eager loading / dataloader | 🔴 | no DB |
| 5.2 | **DB indexes** | Indexed on WHERE / JOIN / ORDER BY columns | 🔴 | tiny dataset |
| 5.3 | **Connection pool** | DB / Redis pool size set & tuned | 🟠 | local single-request |
| 5.4 | **Pagination** | limit/offset or cursor on list APIs | 🔴 | small fixed data |
| 5.5 | **Caching strategy** | Hot data in Redis / memory with TTL | 🟠 | write-heavy |
| 5.6 | **Async I/O** | Non-blocking I/O | 🟠 | trivial scripts |
| 5.7 | **Response compression** | gzip / brotli | 🟠 | internal API |
| 5.8 | **Slow query detection** | Slow query log + alerts above threshold | 🟡 | — |
| 5.9 | **Memory leak prevention** | Close listeners / timers / connections | 🟠 | one-shot batch |
| 5.10 | **Batch processing** | Bulk insert/update over per-row loops | 🟠 | trivial CRUD |

---

## 6. Testing

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 6.1 | **Unit tests** | Core business logic | 🔴 | pure CRUD only |
| 6.2 | **Integration tests** | DB / external API endpoints | 🟠 | — |
| 6.3 | **E2E tests** | Real user-flow automation | 🟡 | internal tools |
| 6.4 | **Coverage** | Core logic 80%+ | 🟠 | side project |
| 6.5 | **Test isolation** | No shared state between tests | 🟠 | — |
| 6.6 | **Mocking strategy** | External APIs mocked; DB real or in-memory | 🟠 | — |
| 6.7 | **TDD** | Write failing test first when fixing bugs | 🟠 | exploratory early |
| 6.8 | **Load testing** | 2–3× expected traffic | 🟡 | MAU < 1k |
| 6.9 | **Contract testing** | Detect breaking API changes | 🟡 | single client |

---

## 7. DevOps / Deployment

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 7.1 | **Secret management** | GitHub Secrets / Vault / SSM; .env not committed | 🔴 | — |
| 7.2 | **Pin dependency versions** | Lock files committed | 🔴 | — |
| 7.3 | **Environment separation** | dev / staging / prod isolated configs | 🟠 | local-only |
| 7.4 | **CI/CD pipeline** | push → test → build → deploy automated | 🟠 | solo / side |
| 7.5 | **Containerization** | Dockerfile / docker-compose | 🟠 | serverless |
| 7.6 | **Rollback strategy** | Prior-version recovery on failed deploy | 🟠 | downtime OK |
| 7.7 | **Blue-green / canary** | Zero-downtime deploys | 🟡 | < 99% SLA |
| 7.8 | **Infrastructure as Code** | Terraform / Pulumi | 🟡 | single server |
| 7.9 | **Build reproducibility** | Same code + deps → same build | 🟠 | — |

---

## 8. Scalability

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 8.1 | **Stateless design** | No server-local state; sessions externalized | 🟡 | single instance |
| 8.2 | **Horizontal scalability** | Adding instances increases throughput | 🟡 | predictable traffic |
| 8.3 | **Async processing** | Long jobs via queue | 🟠 | fast-response only |
| 8.4 | **DB read/write split** | Read replicas | 🔵 | MAU < 100k |
| 8.5 | **CDN** | Static / images via CDN | 🟡 | API only |
| 8.6 | **Event-driven architecture** | Events instead of direct calls | 🔵 | single service |

---

## 9. 12-Factor App

| # | Factor | Description | Grade |
|---|--------|-------------|-------|
| 9.1 | **Config in env vars** | No values in code, all in ENV | 🔴 |
| 9.2 | **Logs to stdout** | Don't write log files; infra collects | 🟠 |
| 9.3 | **Stateless processes** | Restart-safe; no local-file dependence | 🟠 |
| 9.4 | **Explicit dependencies** | requirements.txt / package.json | 🔴 |
| 9.5 | **Dev/prod parity** | Local ≈ staging ≈ prod | 🟠 |

---

## 10. Documentation

| # | Item | Description | Grade | Skippable when |
|---|------|-------------|-------|----------------|
| 10.1 | **README** | Purpose, install, run, env vars | 🔴 | — |
| 10.2 | **API docs** | Swagger / OpenAPI / Postman | 🟠 | internal tools |
| 10.3 | **Architecture diagram** | Service / DB / external deps | 🟠 | trivial single service |
| 10.4 | **ADR** | Architecture Decision Records | 🟡 | team ≤ 2 |
| 10.5 | **Runbook** | Incident response, restart, known issues | 🟠 | solo ops |
| 10.6 | **Changelog** | Per-version changes | 🟠 | internal tools |
