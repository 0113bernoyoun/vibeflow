---
name: vibeflow
description: Java/Spring Boot 프로젝트에서 TDD 기반 설계-구현-검증 파이프라인을 behavior 단위 루프로 실행합니다. planner, builder, verifier 하위 스킬을 순차 오케스트레이션하여 Red-Green-Refactor 사이클을 강제합니다. Use when TDD로 새 기능을 구현하거나, 기존 코드를 리팩토링하거나, behavior 기반 개발이 필요할 때. Triggers on "vibeflow", "TDD로 구현", "behavior 단위 개발", "Red-Green-Refactor".
allowed-tools: Read, Glob, Grep, Bash
---

# VibeFlow: TDD Loop Controller

당신은 사용자의 아이디어를 TDD로 구현하는 전체 공정의 **루프 컨트롤러**입니다.
일직선 파이프라인이 아닌, **behavior 단위 반복**으로 TDD 사이클을 강제합니다.

## Plan Naming

파이프라인 시작 시 유니크한 plan을 생성합니다.

### 이름 생성 규칙
1. 사용자의 요청에서 핵심 키워드 2~4개를 추출합니다.
2. kebab-case로 변환합니다 (한글은 영어로 번역).
3. Plan 경로: `docs/vibe/{slug}/{slug}-plan.md`

예시:
- "장바구니 TDD 검증" → `docs/vibe/cart-tdd-verification/cart-tdd-verification-plan.md`
- "주문 검색 리팩토링" → `docs/vibe/order-search-refactor/order-search-refactor-plan.md`

### 디렉토리 내 산출물 파일

| 파일 | 생성 시점 | 역할 |
|---|---|---|
| `{slug}-spec.md` | Phase 1 (planner) | 요구사항, 전제, 범위, 제약 — "무엇을 왜" |
| `{slug}-plan.md` | Phase 1 (planner) | behavior 명세, 순서, 테스트 전략 — "어떻게" |
| `{slug}-lessons.md` | Phase 3.5 (vibeflow) | 교훈, 발견된 엣지케이스, 프로세스 개선점 |

### 기존 Plan 감지
- 동일 slug 디렉토리가 이미 존재하면 사용자에게 확인:
  - **이어하기**: 기존 {slug}-plan.md를 읽고 미완료 behavior부터 진행
  - **새로 만들기**: slug에 `-v2`, `-v3` 등 suffix 추가

### Sub-skill 호출 시 Plan Path 전달
모든 sub-skill 호출에 `plan-path` 인자를 명시적으로 전달합니다:
- `/planner plan-path=docs/vibe/{slug}/{slug}-plan.md`
- `/builder plan-path=docs/vibe/{slug}/{slug}-plan.md behavior=N`
- `/verifier plan-path=docs/vibe/{slug}/{slug}-plan.md`

## 실행 파이프라인

### Phase 0: 요구사항 명확화

planner를 호출하기 **전에** 사용자의 요청을 검토합니다.

#### 모호성 체크리스트
아래 항목 중 하나라도 해당하면 사용자에게 질문합니다:

- [ ] **대상이 불분명**: 어떤 Entity/도메인을 다루는지 특정할 수 없음
- [ ] **범위가 불분명**: "개선해줘", "리팩토링해줘" 등 구체적 범위 없음
- [ ] **기대 결과가 불분명**: 완료 조건을 정의할 수 없음
- [ ] **여러 해석 가능**: 요청이 2가지 이상의 방향으로 해석됨
- [ ] **기술적 제약 미확인**: DB 스키마 변경, 외부 API 연동 등 확인 필요 사항이 있음

#### 도메인 컨텍스트 확인
사용자에게 **이 작업이 속한 도메인**을 질문합니다 (예: 커머스, 의료, 교육, 복지 등).

1. `skills/planner/references/domains/{domain}.md` 파일이 존재하면 planner에 전달합니다.
2. 파일이 없으면 planner에게 **웹 리서치 후 사용자 확인** 모드로 도메인 컨텍스트를 수집하도록 지시합니다.
3. 도메인이 명확하지 않거나 범용적인 작업이면 도메인 컨텍스트 없이 진행합니다.

#### 질문 방식
- 모호한 항목만 골라서 **최소한의 질문**을 합니다 (한 번에 3개 이하).
- 도메인 질문은 모호성 질문과 함께 한 번에 합니다.
- 명확한 부분은 질문하지 않습니다.
- 모든 항목이 명확하면 질문 없이 Phase 1로 진행합니다.

### Phase 1: Planning (`/planner` 호출)

- slug를 생성하고 `docs/vibe/{slug}/{slug}-plan.md` 경로를 결정합니다.
- Phase 0에서 수집한 **Q&A 결과**(사용자 질문과 응답)가 있으면 planner 호출 시 컨텍스트로 전달합니다.
  - planner가 같은 질문을 다시 하지 않도록, Phase 0에서 해소된 사항을 명시합니다.
- `/planner plan-path=docs/vibe/{slug}/{slug}-plan.md`를 호출하여 {slug}-plan.md를 생성합니다.
- **도메인 컨텍스트**가 있으면 planner 호출 시 함께 전달합니다:
  - 도메인 참조 파일 경로 (있는 경우)
  - 웹 리서치 필요 여부 (참조 파일이 없는 경우)
- planner는 **`{slug}-spec.md`도 함께 생성**합니다:
  - 사용자가 별도 spec 문서를 제공한 경우 → 그것을 `{slug}-spec.md`로 저장
  - 제공하지 않은 경우 → Phase 0 Q&A + Understanding Summary를 `{slug}-spec.md`로 생성
- {slug}-plan.md에는 **행동(behavior) 명세 리스트**가 순서대로 포함됩니다.
- 테스트 코드는 작성하지 않습니다. 행동의 입력/출력/조건만 정의합니다.
- 복잡한 작업일 경우, 반드시 사용자에게 승인을 받은 후 다음 단계로 넘어갑니다.

#### {slug}-plan.md 필드 완전성 검증

Planner가 {slug}-plan.md를 생성한 후, **Phase 1.5로 진행하기 전에** 아래 필수 필드가 모든 behavior에 존재하는지 확인합니다:

| 필수 필드 | 누락 시 행동 |
|---|---|
| `depends_on` | planner에 되돌림 |
| `testability` | planner에 되돌림 |
| `mock 전략` | planner에 되돌림 |
| `size 판정` | planner에 되돌림 |
| `기존 테스트` | planner에 되돌림 |

- **하나라도 누락된 behavior가 있으면** planner를 다시 호출하여 해당 필드를 보완하도록 지시합니다.
- 모든 필드가 채워져 있으면 Phase 1.5로 진행합니다.

### Phase 1.5: Baseline Test Capture (기존 테스트 기준선 확보)

Planning 완료 후, builder 호출 **전에** 기존 테스트 상태를 캡처합니다.

1. {slug}-plan.md의 `Existing Test Baseline` 섹션을 확인합니다.
2. 영향 범위 모듈의 **전체 테스트를 실행**합니다: `./gradlew :모듈명:test`
3. 아래 기준선을 기록합니다:
   ```
   [Global Test Baseline]
   - 모듈: {모듈명}
   - 총 테스트 수: {N}개
   - 결과: BUILD SUCCESSFUL
   - 테스트 파일별 평가:
     - {파일1}: {N}개 (양호/보호)
     - {파일2}: {N}개 (빈약/보강)
     - {파일3}: {N}개 (잘못됨/수정)
   ```
4. **보호 대상 테스트는 Phase 2 전체에서 변경이 금지됩니다.**
5. 테스트 실행이 실패하면 builder를 호출하기 전에 사용자에게 보고합니다.

### Phase 2: Building (behavior별 `/builder` 반복 호출)

**핵심: builder를 behavior 1개 단위로 반복 호출합니다.**

```
for each behavior in {slug}-plan.md:
  /builder plan-path=docs/vibe/{slug}/{slug}-plan.md behavior=N → TDD 사이클 실행
    1. Red: 이 behavior에 대한 테스트 1개 작성
    2. Red 확인: 테스트 실행 → 실패 출력 기록
    3. Green: 최소 구현
    4. Green 확인: 전체 테스트 실행 → 통과 확인
    5. Refactor: 중복 제거, 설계 개선
    6. Refactor 확인: 전체 테스트 실행 → 여전히 통과 확인
  builder 결과 수신 → 중간 보고 확인
  다음 behavior로 진행
```

#### builder 호출 시 전달 내용

각 builder 호출 시 아래 정보를 명시적으로 전달하세요:

1. **현재 behavior 번호와 명세** ({slug}-plan.md에서 해당 항목)
2. **이전까지 완료된 behavior 목록** (누적 컨텍스트)
3. **"이 behavior만 처리하라"는 명시적 지시**
4. **{slug}-plan.md 전체 경로** (참조용)
5. **Global Test Baseline 정보** (기존 테스트 수, 파일별 처리 방침)

#### 중간 보고 확인

각 builder 완료 후 아래를 확인하세요:

- [ ] 테스트가 구현 전에 실패했다는 증거 (실패 출력)
- [ ] 최소 구현 후 통과했다는 증거 (성공 출력)
- [ ] 리팩토링 수행 여부
- [ ] 전체 테스트가 여전히 통과하는지
- [ ] **기존 테스트 관리**: Baseline & Assessment가 기록되어 있는지
- [ ] **보호 대상**: "보호" 등급 테스트가 전체 통과하는지
- [ ] **변경 근거**: 보강/수정/삭제한 테스트에 변경 사유가 기록되어 있는지
- [ ] **테스트 커버리지**: 삭제된 테스트에 대체 테스트가 존재하는지

**증거가 불충분하면 해당 behavior를 다시 시도하세요.**
**보호 대상 테스트가 깨지면 즉시 중단하고 원인을 파악하세요.**
**{slug}-plan.md에 명시되지 않은 기존 테스트 변경이 발견되면 builder에게 되돌리도록 지시하세요.**

### Phase 3: Verifying (`/verifier` 호출)

- `/verifier plan-path=docs/vibe/{slug}/{slug}-plan.md`를 호출합니다.
- **Global Test Baseline 정보를 verifier에 전달합니다** (기존 테스트 파일 목록, 메서드 수, 처리 방침).
- 모든 behavior 구현 완료 후 최종 검증을 수행합니다.
- **TDD 준수 여부 검증**이 핵심 항목에 추가됩니다.
- **기존 테스트 관리 검증**이 핵심 항목에 추가됩니다.
- **빌드 및 컴파일 검증**: 전체 프로젝트가 `./gradlew build`로 정상 빌드되는지 확인합니다.
- 코드 품질, 프로젝트 관례 준수, AI 오염 여부를 검토합니다.

### Phase 3.5: Lessons Learned (교훈 기록)

Verifier의 Verification Report를 수신한 후, **교훈을 추출하여 저장**합니다.

1. Verifier Report의 `### Lessons Learned` 섹션에서 교훈을 추출합니다.
2. `docs/vibe/{slug}/{slug}-lessons.md`에 기록합니다:
   ```
   # Lessons Learned: {slug}

   ## 도메인 교훈
   - {발견된 도메인 엣지케이스, 패턴}

   ## 프로세스 교훈
   - {behavior 크기 판단 오류, spec 모호성 등}

   ## 기술적 교훈
   - {테스트 패턴, 구현 접근법 등}
   ```
3. **도메인 관련 교훈**이 있으면 `skills/planner/references/domains/{domain}.md`에 append합니다.
4. `{slug}-spec.md`에 **발견된 엣지케이스와 변경된 전제**를 반영합니다.

## 운영 원칙

- 모든 의사결정과 진행 상황은 `docs/vibe/{slug}/` 디렉토리에 기록물(.md)로 남깁니다.
- builder를 **한 번만 호출하여 전체를 처리하게 하는 것은 금지**입니다.
- 각 behavior의 TDD 증거가 확보되지 않으면 다음으로 넘어가지 마세요.
- **언어**: Java/Spring Boot 환경 기준입니다. Kotlin이 아닙니다.

## Gotchas

1. **builder를 한 번만 호출하여 전체를 처리하려는 경향**: 반드시 behavior 1개 단위로 반복 호출해야 합니다. 한 번에 전체를 처리하면 TDD 사이클이 무효화됩니다.
2. **{slug}-plan.md 필드 누락을 확인하지 않고 Phase 1.5로 넘어가는 실수**: Phase 1 완료 후 반드시 필드 완전성 검증을 수행하세요. depends_on, testability, mock 전략, size 판정, 기존 테스트가 모두 채워져야 합니다.
3. **기존 Plan이 존재할 때 사용자 확인 없이 덮어쓰는 실수**: 동일 slug 디렉토리가 존재하면 반드시 사용자에게 "이어하기"와 "새로 만들기" 중 선택을 요청하세요.
4. **builder 중간 보고의 TDD 증거를 검증하지 않고 다음 behavior로 넘어가는 실수**: Red 실패 출력, Green 통과 출력, Refactor 통과 출력이 모두 있어야 다음 behavior로 진행할 수 있습니다.

