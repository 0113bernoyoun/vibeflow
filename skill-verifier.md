# Skill Verifier: Claude Code 스킬 검증 방법론

Claude Code 스킬(SKILL.md)의 품질을 검증하는 재사용 가능한 체크리스트입니다.

## 검증 소스

이 방법론은 아래 3개 소스를 종합하여 구성되었습니다:

| 소스 | 유형 | 검증 범위 |
|---|---|---|
| **Pulser** (whynowlab/pulser) | 정적 분석 도구, 8개 규칙 | 구조적 검증 (frontmatter, 줄 수, 파일 구조) |
| **Anthropic 공식 문서** (code.claude.com, agentskills.io, platform.claude.com) | 공식 스펙 + Best Practices | 필드 규칙, 호출 제어, 도구 권한, 명명 규칙 |
| **Anthropic 내부 사용 경험** (블로그 포스트) | 실전 노하우 | Progressive Disclosure, Gotchas, 스킬 유형 분류, 안티패턴 |

---

## Phase 1: 구조 검증 (Structural)

### 1.1 Frontmatter 필수 필드

| 필드 | 규칙 | 심각도 |
|---|---|---|
| `name` | 필수. 소문자 영숫자 + 하이픈만. 64자 이내. 디렉토리명과 일치. 하이픈으로 시작/끝 불가. 연속 하이픈(`--`) 불가. | ERROR |
| `description` | 필수. 100~500자. 3인칭. "Use when" 또는 "Triggers on" 트리거 언어 포함. 무엇을 하는지 + 언제 사용하는지 모두 기술. | ERROR |

### 1.2 Frontmatter 권장 필드

| 필드 | 용도 | 심각도 |
|---|---|---|
| `allowed-tools` | 최소 권한 원칙. 스킬이 실제 필요한 도구만 선언. | WARNING |
| `disable-model-invocation` | `true`면 Claude 자동 호출 차단. 하위 스킬에 권장. | WARNING |
| `user-invocable` | `false`면 사용자 `/` 메뉴에서 숨김. 배경 지식 스킬에 사용. | INFO |
| `argument-hint` | 자동완성 시 표시할 인자 힌트. 예: `[plan-path] [behavior]` | INFO |
| `model` | 스킬 활성화 시 사용할 모델. | INFO |
| `context` | `fork`으로 설정하면 서브에이전트 컨텍스트에서 실행. | INFO |

### 1.3 파일 크기

| 기준 | 심각도 |
|---|---|
| 400줄 이상 | WARNING — 500줄 제한에 접근 중 |
| **500줄 이상** | **ERROR** — 참조 파일로 분리 필수 |
| 200줄 이상 & 참조 파일 없음 | WARNING — `references/`, `examples/`, `scripts/` 활용 권장 |

### 1.4 디렉토리 구조

```
skill-name/
├── SKILL.md           (필수, 500줄 이하)
├── references/        (200줄 초과 시 권장)
├── scripts/           (실행 가능 코드)
├── assets/            (템플릿, 리소스)
└── examples/          (사용 예시)
```

- 참조 파일은 SKILL.md에서 **1단계 깊이**까지만 (깊게 중첩된 참조 체인 금지)
- 100줄 이상 참조 파일은 목차 포함

---

## Phase 2: Description 품질 검증

### 2.1 트리거 언어 확인

Description에 아래 패턴 중 하나 이상 포함되어야 함:
- `"Use when"`
- `"Triggers on"`
- `"Invoke when"`

### 2.2 내용 완결성

- [ ] **무엇을 하는지** (What) 기술됨
- [ ] **언제 사용하는지** (When) 기술됨
- [ ] 구체적 **트리거 키워드** 포함 (따옴표로 감싼 키워드)
- [ ] **3인칭**으로 작성 ("I can help" ✗, "Processes files" ✓)

### 2.3 트리거 키워드 충돌

같은 프로젝트 내 다른 스킬과 트리거 키워드가 겹치면 Claude가 잘못된 스킬을 호출할 수 있음.

- [ ] 각 스킬의 description에서 따옴표로 감싼 키워드를 추출
- [ ] 동일 키워드가 2개 이상의 스킬에 존재하면 WARNING

---

## Phase 3: 도구 권한 검증 (allowed-tools)

### 3.1 최소 권한 원칙

스킬 유형별 권장 도구:

| 스킬 유형 | 권장 allowed-tools |
|---|---|
| 분석/탐색 (read-only) | `Read, Grep, Glob` |
| 연구/조사 | `Read, WebSearch, WebFetch` |
| 구현/실행 | `Read, Write, Edit, Bash` |
| 생성/작성 | `Read, Write, Edit` |
| 참조/지식 | `Read` |

### 3.2 위험 도구 확인

- [ ] read-only 스킬에 `Write`, `Edit`, `Bash`가 포함되어 있지 않은가?
- [ ] 분석 스킬에 `Bash`가 포함되어 있지 않은가?

---

## Phase 4: 콘텐츠 품질 검증

### 4.1 Gotchas 섹션

- [ ] `## Gotchas` 헤딩이 존재하는가?
- [ ] 번호 매긴 목록(`1. `)이 1개 이상 있는가?
- [ ] 실제 실패 사례 기반인가? (일반적 주의사항이 아닌 구체적 실패 패턴)

### 4.2 Don't State the Obvious

- [ ] Claude가 이미 아는 일반 코딩 지식을 반복하지 않는가?
- [ ] 스킬 고유의 guardrail과 도메인 지식에 집중하는가?

### 4.3 Progressive Disclosure 활용

- [ ] SKILL.md 본문에 긴 템플릿/형식이 인라인되어 있지 않은가?
- [ ] 상세 참조 자료가 `references/`로 분리되어 있는가?
- [ ] Claude에게 참조 파일 위치를 알려주고 있는가?

### 4.4 Railroading 판단

| 영역 | 엄격함 적절? | 유연함 적절? |
|---|---|---|
| 프로세스 순서 (TDD 단계 등) | ✓ 엄격함 필요 | |
| 출력 형식 (보고서, 템플릿) | ✓ 엄격함 필요 | |
| 구현 방법 (프레임워크, 도구) | | ✓ 유연함 필요 |
| 네이밍/컨벤션 | ✓ 프로젝트 규칙이면 엄격 | ✓ 일반 코딩이면 유연 |

---

## Phase 5: 호출 제어 검증

### 5.1 호출 매트릭스

| 설정 | 사용자 호출 | Claude 자동 호출 | 적합한 스킬 |
|---|---|---|---|
| (기본값) | ✓ | ✓ | 메인 진입점 스킬 |
| `disable-model-invocation: true` | ✓ | ✗ | 하위 스킬 (상위 스킬만 호출) |
| `user-invocable: false` | ✗ | ✓ | 배경 지식, 자동 감지용 |

### 5.2 하위 스킬 보호

상위 스킬이 호출하는 하위 스킬에는:
- [ ] `disable-model-invocation: true` 설정됨
- [ ] Description에 "상위 스킬에서 호출됨" 명시

---

## Phase 6: 스킬 유형 분류

스킬이 아래 유형 중 하나에 명확히 속하는지 확인. 여러 유형에 걸치면 분리 검토.

| 유형 | 설명 | 예시 |
|---|---|---|
| Library & API Reference | 라이브러리/SDK 사용법, 주의사항 | 내부 SDK 가이드 |
| Product Verification | 코드 동작 검증, 테스트 자동화 | E2E 테스트 드라이버 |
| Data Fetching & Analysis | 데이터/모니터링 스택 연결 | Grafana 조회 |
| Business Process Automation | 반복 워크플로 자동화 | 스탠드업 자동 작성 |
| Code Scaffolding & Templates | 보일러플레이트 생성 | 새 서비스 스캐폴드 |
| Code Quality & Review | 코드 품질 강제, 리뷰 | 코드 스타일 검사 |
| CI/CD & Deployment | 빌드, 배포, PR 관리 | PR 모니터링 |
| Runbooks | 증상 → 조사 → 보고서 | 장애 대응 |
| Infrastructure Operations | 인프라 유지보수, 정리 | 리소스 정리 |

---

## 검증 보고서 템플릿

```markdown
# Skill Verification Report: {스킬명}

## 요약
- 검증일: {날짜}
- 파일: {경로}
- 줄 수: {N}줄

## 결과

| Phase | 항목 | 결과 | 비고 |
|---|---|---|---|
| 구조 | name 필드 | PASS/FAIL | |
| 구조 | description 필드 | PASS/FAIL | |
| 구조 | 줄 수 제한 | PASS/WARN/FAIL | |
| 구조 | 디렉토리 구조 | PASS/WARN | |
| Description | 트리거 언어 | PASS/FAIL | |
| Description | 내용 완결성 | PASS/FAIL | |
| Description | 키워드 충돌 | PASS/WARN | |
| 도구 권한 | allowed-tools 선언 | PASS/WARN | |
| 도구 권한 | 최소 권한 준수 | PASS/WARN | |
| 콘텐츠 | Gotchas 섹션 | PASS/WARN | |
| 콘텐츠 | Progressive Disclosure | PASS/WARN | |
| 호출 제어 | 호출 매트릭스 적절성 | PASS/WARN | |

## 판정
**PASS** / **WARN (N건)** / **FAIL (N건)**
```

---

## 참고 자료

- [Pulser — Claude Code Skill Diagnostic Tool](https://github.com/whynowlab/pulser)
- [Claude Code Skills 공식 문서](https://code.claude.com/docs/en/skills)
- [Agent Skills Specification](https://agentskills.io/specification)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Anthropic 공식 스킬 예시](https://github.com/anthropics/skills)
- Anthropic 내부 스킬 운영 경험 (블로그 포스트)
