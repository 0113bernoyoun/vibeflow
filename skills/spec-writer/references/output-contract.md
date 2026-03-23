# Spec Output Contract

spec-writer가 생성하는 spec.md의 구조 계약입니다.
planner(Mode A)가 이 계약에 따라 spec.md를 파싱하고 검증합니다.

## 필수 섹션

모든 spec.md에 반드시 포함되어야 하는 섹션입니다.

```markdown
# Spec: {제목}

## Meta
- domain: {도메인명 또는 "general"}
- scope: {full | standard | light}
- status: {draft | reviewed | approved}
- created: {날짜}

## Overview
{목적, 비즈니스 동기, 성공 기준}

## Scope
### In Scope
{포함 범위}
### Out of Scope
{제외 범위}
### Deferred
{다음 단계로 미루는 것}

## Requirements
### REQ-001: {요구사항 제목}
- **statement**: {EARS 표기법 — ears-guide.md 참조}
- **priority**: must | should | could
- **acceptance**: {수용 기준 — 테스트 가능한 형태}
- **related**: [REQ-NNN, ...] 또는 없음

## Business Rules
### BR-001: {규칙명}
- **조건**: {언제 적용되는가}
- **결과**: {어떤 결과가 되는가}

## Edge Cases
### EC-001: {엣지케이스명}
- **상황**: {언제 발생}
- **기대 동작**: {시스템이 어떻게 반응해야 하는지}
- **related**: [REQ-NNN] (관련 요구사항)

## Assumptions
| ID | 가정 | 확인 상태 | 확인 방법 |
|---|---|---|---|
| ASM-001 | {내용} | confirmed / unconfirmed | {사용자 확인 / 코드 확인} |

## Open Questions
| ID | 질문 | 영향 범위 | 차단 여부 |
|---|---|---|---|
| OQ-001 | {미결정 사항} | {영향받는 REQ-NNN} | blocking / non-blocking |
```

## 선택 섹션

Phase 3-3에서 필요하다고 판단된 섹션만 추가합니다.

| 섹션 | 추가 조건 |
|---|---|
| `## API Specification` | API가 필요할 때 |
| `## UI Specification` | 화면 인터랙션이 있을 때 |
| `## Data Model` | Entity/테이블 변경이 있을 때 |
| `## State Transitions` | 상태 전이가 있을 때 (Mermaid stateDiagram 포함) |
| `## Integration` | 외부 시스템 연동이 있을 때 |
| `## Calculation Logic` | 금액/수량 계산이 있을 때 |
| `## Batch / Async Processing` | 배치/비동기 처리가 필요할 때 |
| `## Authorization Model` | 역할별 접근 제어가 필요할 때 |
| `## Migration Plan` | 데이터 마이그레이션이 동반될 때 |
| `## Non-Functional Requirements` | FURPS+ 점검에서 해당 항목이 있을 때 |
| `## Domain Context` | 도메인 리서치 결과가 있을 때 |
| `## Codebase Context` | 코드베이스 탐색 결과가 있을 때. 유사 기능 발견 시 **품질 등급(신뢰/참고/경고)**을 포함해야 함 |

## ID 체계

| 접두어 | 용도 | 예시 |
|---|---|---|
| `REQ-NNN` | 기능 요구사항 | REQ-001, REQ-002 |
| `BR-NNN` | 비즈니스 규칙 | BR-001 |
| `EC-NNN` | 엣지케이스 | EC-001 |
| `ASM-NNN` | 가정 | ASM-001 |
| `OQ-NNN` | 미결정 사항 | OQ-001 |

## planner 소비 규칙

planner가 spec.md를 받았을 때의 행동:

### Mode A (spec 있음)
1. **구조 검증**: 필수 섹션 존재 확인
2. **ID 검증**: 모든 REQ에 ID, statement, priority, acceptance가 있는지 확인
3. **코드 정합성**: REQ별로 기존 코드와 충돌하지 않는지 탐색
4. **보강**: 누락된 엣지케이스, 미검증된 가정을 발견하면 사용자에게 보고
5. **behavior 매핑**: REQ-NNN → Behavior N 매핑을 생성
6. **OQ 처리**: blocking OQ가 있으면 사용자에게 해소 요청, non-blocking은 behavior에 `[미결정]` 태그

### Mode B (spec 없음)
현재 planner 흐름 유지 (Q&A → Understanding Summary → 간이 spec → behaviors)
