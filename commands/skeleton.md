---
description: ai-project-audit 섹션 0(프로젝트 골조)만 적용해 새 프로젝트 뼈대를 잡는다
---

`project-audit` 스킬을 발동시키되, **섹션 0(프로젝트 골조)에만** 한정합니다.

순서:
1. 서비스 프로필 받기 (없으면 한 번에 묻기).
2. `skills/project-audit/references/audit-spec.md`의 0.1 ~ 0.12 항목을 프로필에 맞게 구체화:
   - 디렉토리 구조 (레이어/피처/도메인 중 추천)
   - 네이밍 컨벤션
   - Git 브랜치 전략
   - 커밋 메시지 규칙
   - 린터·포맷터·pre-commit
   - README / .env.example 템플릿
   - 에러 코드 체계
   - API 계약 방식
   - 로컬 원터치 실행 명령
3. 권장안을 실제 파일로 생성 (README.md, .env.example, 린터 설정, pre-commit 훅, 디렉토리 뼈대).
4. **기능 코드는 짜지 않는다.** 골조만.

대상 프로젝트 경로: $ARGUMENTS
