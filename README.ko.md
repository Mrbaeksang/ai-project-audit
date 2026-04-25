# ai-project-audit

> **Karpathy의 4가지 코딩 원칙 + 10개 섹션 프로덕션 감사**를 하나의 Claude Code 플러그인으로. 서비스 프로필만 알려주면, 뭐가 빠졌는지 알려줍니다.

[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-orange)](https://docs.anthropic.com/claude-code) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) [![Multilingual](https://img.shields.io/badge/i18n-EN%20%7C%20한국어-green)](README.md)

🇬🇧 **[English README](README.md)**

---

## 무엇을 하는가

두 가지 행동 규약을 Claude Code에 한 번에 주입:

1. **Karpathy Discipline** — 코드 짜기 전 생각 · 단순함 우선 · 외과적 수정 · 검증 가능한 목표. drive-by 리팩토링·과설계 차단. ([출처](https://x.com/karpathy/status/2015883857489522876))
2. **Project Audit** — 10줄짜리 서비스 프로필(MAU, 단계, 민감도 등)을 받아서 0~10 섹션 체크리스트(OWASP / SOLID / ACID / 12-Factor / …)를 등급(🔴🟠🟡🔵⚪)별로 필터링하고 항목마다 `✅ ❌ ⚠️ ⏭️`로 결과를 뽑음 — 위험도 순 정렬.

**다국어**: 스킬은 영어로 작성됐지만 **사용자 언어로 응답**. 한국어로 물으면 한국어, 영어면 영어. 설정 불필요.

---

## 설치 (30초)

```text
/plugin marketplace add Mrbaeksang/ai-project-audit
/plugin install ai-project-audit@mrbaeksang-marketplace
```

또는 단일 파일 (플러그인 안 써도 됨):

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/Mrbaeksang/ai-project-audit/main/CLAUDE.md
```

---

## 사용

```text
/audit ./src         # 전체 감사 — 서비스 프로필 묻고 0~10 섹션 점검
/skeleton .          # 섹션 0만 — 기능 코드 짜기 전 골조 잡기
```

또는 자연어로: *"OWASP 기준으로 점검해줘"*, *"출시 전 리뷰"*, *"프로젝트 골조 잡아줘"*, *"12-Factor 따르고 있는지 체크"*.

### 결과 예시

```
✅ 1.1 Health endpoint — src/health.controller.ts:12
❌ 2.10 CORS — src/main.ts:8 (wildcard `*` + credentials)
   수정: app.enableCors({ origin: ['https://app.example.com'], credentials: true })
⚠️ 5.2 DB 인덱스 — users.email 인덱스 없음 (로그인 핫패스)
⏭️ 8.4 read replica — MAU 50k는 단일 DB로 충분

이것부터 고쳐라 (위험도 순):
1. ❌ 2.10 CORS  ← 보안
2. ❌ 1.4 타임아웃 ← 운영
...
```

---

## 들어있는 것

| 파일 | 역할 |
|------|------|
| **[SPEC.md](SPEC.md)** | 전체 0~10 체크리스트 (AI가 참조하는 source of truth, 영어) |
| **[CLAUDE.md](CLAUDE.md)** | 행동 규약 (Karpathy 4 + 감사 트리거) |
| **[EXAMPLES.md](EXAMPLES.md)** | 사용 시나리오 |
| `skills/karpathy-discipline/` | 코드 편집 시 자동 발동 |
| `skills/project-audit/` | 감사·리뷰 키워드에 자동 발동 |
| `commands/audit.md`, `commands/skeleton.md` | `/audit`, `/skeleton` |

### 10개 감사 섹션

`0` 골조 · `1` 운영 안정성 · `2` 보안(OWASP) · `3` 아키텍처(SOLID) · `4` 데이터(ACID) · `5` 성능 · `6` 테스트 · `7` DevOps · `8` 확장성 · `9` 12-Factor · `10` 문서화

---

## 어떻게 *당신*에게 맞는 항목만 골라내는가

감사할 때 스킬이 다음을 묻습니다 (한 번):

```yaml
service_type: REST API           # 또는 GraphQL / 모놀리스 / 마이크로서비스 / CLI / 배치
stage: pre-launch                # 또는 planning / building / in-production
expected_mau: 50000
team_size: 3
data_sensitivity: medium         # PII
uptime_target: 99.9%
deployment: k8s
exposure: internet-facing
```

이 프로필 기준으로 각 항목이 등급 평가됨 — 🟡 중규모 항목은 혼자 운영이면 스킵, 🔵 대규모 항목은 MAU 100k 미만이면 스킵, 등. **자기에게 필요한 것만 보임.**

---

## 라이선스

MIT — [LICENSE](LICENSE)

## 영감

[Andrej Karpathy의 LLM 코딩 함정](https://x.com/karpathy/status/2015883857489522876)과 [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills). 기반: [OWASP Top 10](https://owasp.org/www-project-top-ten/), [12-Factor App](https://12factor.net/).
