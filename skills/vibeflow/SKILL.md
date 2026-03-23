---
name: vibeflow
description: TDD 기반 설계-구현-검증 파이프라인을 behavior 단위 루프로 실행합니다. planner, builder, verifier 하위 스킬을 순차 오케스트레이션하여 Red-Green-Refactor 사이클을 강제합니다. 빌드 도구(Gradle, Maven, npm 등)를 자동 감지하여 다양한 기술 스택을 지원합니다. Use when TDD로 새 기능을 구현하거나, 기존 코드를 리팩토링하거나, behavior 기반 개발이 필요할 때. Triggers on "vibeflow", "TDD로 구현", "behavior 단위 개발", "Red-Green-Refactor".
allowed-tools: Read, Write, Glob, Grep, Bash
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

### Phase -1: 워크트리 생성 (Git 격리)

파이프라인의 모든 코드 변경은 **격리된 워크트리**에서 진행합니다.

#### 워크트리 경로 규칙

워크트리는 **현재 레포지토리의 형제 디렉토리**에 생성합니다. 레포지토리보다 상위로 올라가거나 무관한 경로에 생성하는 것은 **금지**입니다.

```bash
# 1. 레포지토리 루트와 이름을 확인합니다
REPO_ROOT=$(git rev-parse --show-toplevel)
REPO_NAME=$(basename "$REPO_ROOT")
PARENT_DIR=$(dirname "$REPO_ROOT")

# 2. 워크트리 경로를 결정합니다
WORKTREE_PATH="${PARENT_DIR}/.vibeflow-worktrees/${REPO_NAME}/${slug}"

# 3. 브랜치를 생성하고 워크트리를 초기화합니다
git worktree add "$WORKTREE_PATH" -b "vibeflow/${slug}"
```

**경로 예시:**
- 레포가 `/Users/dev/my-project`이면
- 워크트리는 `/Users/dev/.vibeflow-worktrees/my-project/{slug}`에 생성
- **절대로** `/Users/.vibeflow-worktrees/...`나 `/tmp/...` 등 상위/무관 경로에 생성하지 않음

#### 워크트리 생성 후

1. 이후 모든 Phase는 `$WORKTREE_PATH` 안에서 실행됩니다.
2. 메인 브랜치는 파이프라인 완료 전까지 변경되지 않습니다.
3. sub-skill 호출 시 워크트리 경로를 명시적으로 전달합니다.

**워크트리가 제공하는 것:**
- **격리**: 파이프라인 중 코드 변경이 메인 브랜치에 영향을 주지 않음
- **롤백**: 파이프라인 실패 시 워크트리를 폐기하면 모든 코드 변경이 원복됨
- **병렬 작업**: 다른 작업을 메인 브랜치에서 동시에 진행 가능

### Phase 0: 요구사항 명확화

planner를 호출하기 **전에** 사용자의 요청을 검토합니다.

#### 스펙 문서 확인

사용자에게 **스펙 문서 존재 여부**를 질문합니다:

1. **"이미 있다"** → 경로를 받아 `{slug}-spec.md`로 복사합니다. **output-contract 준수 여부를 확인**합니다:
   - 필수 섹션(Meta, Overview, Scope, Requirements 등)이 존재하고 REQ-NNN ID가 있으면 → planner **Mode A**로 전달
   - 자유 형식 문서이면 → 사용자에게 선택지 제시: **(a)** spec-writer로 변환 (기존 내용을 참고하여 output-contract 형식으로 재작성), **(b)** Mode B로 진행 (planner가 이 문서를 참고 자료로 활용)
2. **"없다, 만들고 싶다"** → `/spec-writer output-path=docs/vibe/{slug}/{slug}-spec.md`를 호출합니다. 완료 후 planner에 **Mode A**로 전달합니다. **→ 모호성 체크리스트와 도메인 컨텍스트 확인을 건너뜁니다** (spec-writer가 인터뷰와 리서치를 이미 수행했으므로).
3. **"필요 없다"** → planner에 **Mode B (spec 없음)**로 전달합니다. planner가 간이 spec을 자체 생성합니다.

**spec-writer가 생성한 spec.md는 output-contract를 준수**하므로 planner가 파싱/검증할 수 있습니다.

#### 모호성 체크리스트 (경로 3 전용, 경로 1-Mode A는 planner가 검증, 경로 2는 건너뜀)
아래 항목 중 하나라도 해당하면 사용자에게 질문합니다:

- [ ] **대상이 불분명**: 어떤 Entity/도메인을 다루는지 특정할 수 없음
- [ ] **범위가 불분명**: "개선해줘", "리팩토링해줘" 등 구체적 범위 없음
- [ ] **기대 결과가 불분명**: 완료 조건을 정의할 수 없음
- [ ] **여러 해석 가능**: 요청이 2가지 이상의 방향으로 해석됨
- [ ] **기술적 제약 미확인**: DB 스키마 변경, 외부 API 연동 등 확인 필요 사항이 있음

#### 프로젝트 빌드 환경 감지

프로젝트 루트에서 빌드 도구를 자동 감지하고, 모든 하위 스킬에 전달할 명령어를 결정합니다.

**감지 순서:**
1. `gradlew` 또는 `build.gradle` 존재 → Gradle
2. `pom.xml` 존재 → Maven
3. `package.json` 존재 → npm/Node.js
4. `Makefile` 존재 → Make
5. 감지 실패 → 사용자에게 질문

**명령어 매핑:**

| 논리적 명령 | Gradle | Maven | npm |
|---|---|---|---|
| `{test_all}` | `./gradlew test` | `mvn test` | `npm test` |
| `{test_single}` | `./gradlew test --tests "{class.method}"` | `mvn test -Dtest={class}#{method}` | `npm test -- --grep "{pattern}"` |
| `{test_module}` | `./gradlew :{module}:test` | `mvn test -pl {module}` | `npm test --workspace={module}` |
| `{compile}` | `./gradlew compileJava` | `mvn compile` | `npm run build` |
| `{build}` | `./gradlew build` | `mvn package` | `npm run build` |

감지된 명령어는 **모든 sub-skill 호출 시 함께 전달**합니다.

#### 도메인 컨텍스트 확인 (경로 3 전용, 경로 2는 건너뜀)
사용자에게 **이 작업이 속한 도메인**을 질문합니다 (예: 커머스, 의료, 교육, 복지 등).

1. `skills/planner/references/domains/{domain}.md` 파일이 존재하면 planner에 전달합니다.
2. 파일이 없으면 planner에게 **웹 리서치 후 사용자 확인** 모드로 도메인 컨텍스트를 수집하도록 지시합니다.
3. 도메인이 명확하지 않거나 범용적인 작업이면 도메인 컨텍스트 없이 진행합니다.

#### 질문 방식
- 모호한 항목만 골라서 **최소한의 질문**을 합니다 (한 번에 3개 이하).
- 도메인 질문은 모호성 질문과 함께 한 번에 합니다.
- 명확한 부분은 질문하지 않습니다.
- 모든 항목이 명확하면 질문 없이 Phase 1로 진행합니다.

#### Phase 0 경로별 실행 요약

| 단계 | 경로 1 (이미 있다) | 경로 2 (spec-writer) | 경로 3 (필요 없다) |
|---|---|---|---|
| 스펙 문서 확인 | ✓ (포맷 검증) | ✓ (spec-writer 호출) | ✓ |
| 모호성 체크리스트 | planner가 검증 | **건너뜀** | ✓ |
| 빌드 환경 감지 | ✓ | ✓ | ✓ |
| 도메인 컨텍스트 | planner가 spec에서 추출 | **건너뜀** | ✓ |

### Phase 1: Planning (`/planner` 호출)

- slug를 생성하고 `docs/vibe/{slug}/{slug}-plan.md` 경로를 결정합니다.
- Phase 0에서 수집한 **Q&A 결과**(사용자 질문과 응답)가 있으면 planner 호출 시 컨텍스트로 전달합니다.
  - planner가 같은 질문을 다시 하지 않도록, Phase 0에서 해소된 사항을 명시합니다.
- `/planner plan-path=docs/vibe/{slug}/{slug}-plan.md`를 호출하여 {slug}-plan.md를 생성합니다.
- **도메인 컨텍스트**가 있으면 planner 호출 시 함께 전달합니다:
  - 도메인 참조 파일 경로 (있는 경우)
  - 웹 리서치 필요 여부 (참조 파일이 없는 경우)
- **planner 모드** (Phase 0 스펙 문서 확인 결과에 따라):
  - **Mode A (spec 있음)**: `{slug}-spec.md` 경로를 planner에 전달합니다. planner는 spec을 검증/보강한 후 behavior를 설계합니다. 도메인 리서치와 Critic Loop는 spec에서 다룬 범위를 건너뛰고, **spec에서 다루지 않은 기술적 정합성**에 집중합니다.
  - **Mode B (spec 없음)**: planner가 Q&A + Understanding Summary를 기반으로 간이 `{slug}-spec.md`를 자체 생성한 후 behavior를 설계합니다.
- {slug}-plan.md에는 **행동(behavior) 명세 리스트**가 순서대로 포함됩니다.
- 테스트 코드는 작성하지 않습니다. 행동의 입력/출력/조건만 정의합니다.
- 복잡한 작업일 경우, 반드시 사용자에게 승인을 받은 후 다음 단계로 넘어갑니다.

#### {slug}-plan.md 필드 완전성 검증

Planner가 {slug}-plan.md를 생성한 후, **Phase 1.5로 진행하기 전에** 아래 필수 필드가 모든 behavior에 존재하는지 확인합니다:

| 필수 필드 | 누락 시 행동 |
|---|---|
| `spec-ref` | Mode A이면 planner에 되돌림, Mode B이면 "N/A" 허용 |
| `depends_on` | planner에 되돌림 |
| `testability` | planner에 되돌림 |
| `mock 전략` | planner에 되돌림 |
| `size 판정` | planner에 되돌림 |
| `기존 테스트` | planner에 되돌림 |

- **하나라도 누락된 behavior가 있으면** planner를 다시 호출하여 해당 필드를 보완하도록 지시합니다.
- **Mode A에서 spec-ref가 "N/A"인 behavior가 있으면**: spec.md에 대응하는 REQ가 없는 것이므로, 해당 behavior가 spec 범위 밖인지 planner에 확인을 요청합니다.
- 모든 필드가 채워져 있으면 Phase 1.5로 진행합니다.

### Phase 1.5: Baseline Test Capture (기존 테스트 기준선 확보)

Planning 완료 후, builder 호출 **전에** 기존 테스트 상태를 캡처합니다.

1. {slug}-plan.md의 `Existing Test Baseline` 섹션을 확인합니다.
2. 영향 범위 모듈의 **전체 테스트를 실행**합니다: `{test_module}` 또는 `{test_all}`
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

#### Mid-Pipeline Spec 대응

Builder가 TDD 사이클 중 spec 문제를 보고하면 **파이프라인을 일시 중단**하고 아래 절차를 따릅니다.

**`[Ambiguous Spec]` — 현재 behavior의 명세가 모호한 경우:**

1. 파이프라인 일시 중단
2. Builder가 보고한 모호 지점을 사용자에게 전달
3. 사용자 답변을 받아 `{slug}-spec.md`와 `{slug}-plan.md`의 해당 behavior를 업데이트
4. 동일 behavior에 대해 builder 재호출

**`[Spec Conflict]` — 현재 구현이 미래 behavior의 명세와 충돌하는 경우:**

1. 파이프라인 일시 중단
2. 사용자에게 충돌 내용을 보고하고 선택지를 제시:
   - **A. 부분 재설계**: 충돌하는 behavior들만 planner에 재호출하여 수정 (완료된 behavior는 유지)
   - **B. 현행 유지**: 충돌을 감수하고 진행 (사용자 판단으로 충돌이 허용 가능한 경우)
   - **C. 전체 재설계**: 남은 behavior 전체를 planner에 재호출
3. 사용자 선택에 따라 진행

**`[Missing Behavior]` — 구현 중 계획에 없는 behavior가 필요함을 발견한 경우:**

1. 파이프라인 일시 중단
2. 사용자에게 누락된 behavior를 보고
3. 승인 시 planner에 **추가 behavior만** 설계하도록 재호출
4. 추가된 behavior를 현재 위치에 삽입하고 순서 재조정
5. 추가된 behavior부터 builder 진행

**부분 재설계 시 {slug}-plan.md 관리:**

- 완료된 behavior (1..N)은 **변경하지 않음**
- 재설계 대상 behavior에 `[Revised]` 태그를 추가하여 원본과 구분
- 추가된 behavior는 기존 번호 사이에 `N-1`, `N-2` 형태로 삽입하거나, 끝에 추가 후 순서를 재지정

### Phase 3: Verifying (`/verifier` 호출)

- `/verifier plan-path=docs/vibe/{slug}/{slug}-plan.md`를 호출합니다.
- **Global Test Baseline 정보를 verifier에 전달합니다** (기존 테스트 파일 목록, 메서드 수, 처리 방침).
- 모든 behavior 구현 완료 후 최종 검증을 수행합니다.
- **TDD 준수 여부 검증**이 핵심 항목에 추가됩니다.
- **기존 테스트 관리 검증**이 핵심 항목에 추가됩니다.
- **빌드 및 컴파일 검증**: 전체 프로젝트가 `{build}`로 정상 빌드되는지 확인합니다.
- 코드 품질, 프로젝트 관례 준수, AI 오염 여부를 검토합니다.

### Phase 3-R: Revision (Requires Revision 대응)

Verifier가 **"Requires Revision"**을 발행하면 이 단계가 활성화됩니다.

#### 실패 분류 및 복구 경로

vibeflow는 Verification Report의 실패 항목을 분류하여 복구 경로를 결정합니다:

| 분류 | 실패 유형 | 복구 경로 |
|---|---|---|
| **A. Builder 수정 가능** | TDD 미준수, 빌드/컴파일 실패, 보호 테스트 변경, 무단 테스트 변경, 대체 없는 삭제 | 사용자에게 알린 후 → builder 재호출 → verifier 재호출 |
| **B. Planner 재설계 필요** | 스펙-코드 불일치 (스펙이 틀림), 구조적 결함 | 사용자에게 판단 요청 → planner 재호출 → Phase 1부터 재진행 |
| **C. 사용자 판단 필요** | 스펙-코드 불일치 (어느 쪽이 틀린지 불분명), 복합 실패 | 사용자에게 전체 보고 → 방향 지시 대기 |

#### 분류 기준

- Verification Report에 **"구조적 결함"** 또는 **"planner부터 재시작 권고"**가 있으면 → **B**
- **스펙-코드 불일치**에서 verifier가 "구현이 틀림"으로 판단했으면 → **A**
- **스펙-코드 불일치**에서 verifier가 "스펙이 틀림" 또는 "판단 불가"로 판단했으면 → **B** 또는 **C**
- 그 외 실패 → **A**

#### Revision 루프

```
revision_count = 0
MAX_REVISION = 3

while verifier_result == "Requires Revision" AND revision_count < MAX_REVISION:
    revision_count += 1

    분류 A (builder 수정 가능):
        사용자에게 알림: "Revision {revision_count}/3: {실패 요약}"
        실패한 behavior만 builder 재호출 (수정 지시 포함)
        verifier 재호출

    분류 B (planner 재설계):
        사용자에게 판단 요청
        승인 시 → planner 재호출 → Phase 1부터 재진행
        (revision_count는 유지)

    분류 C (사용자 판단):
        사용자에게 전체 보고 + 선택지 제시
        사용자 지시에 따라 A 또는 B 경로 진행

if revision_count >= MAX_REVISION AND verifier_result == "Requires Revision":
    파이프라인 중단
    사용자에게 전체 실패 보고서 전달
    "3회 revision 후에도 미해결. 수동 검토가 필요합니다."
```

#### Builder 재호출 시 전달 내용

1. **실패한 behavior 번호와 실패 사유** (Verification Report에서 추출)
2. **구체적 수정 지시** ("TDD 증거를 다시 수집하라", "보호 테스트를 되돌려라" 등)
3. **"해당 behavior만 수정하라"는 명시적 지시** (다른 behavior를 건드리지 않도록)
4. **현재 revision 횟수** (builder가 긴급도를 인지하도록)

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

### Phase 4: Commit & Complete (커밋 및 완료)

Verifier가 **"Safe to Merge"**를 발행하고 Lessons Learned가 기록된 후 실행합니다.

1. 워크트리 내 모든 변경 사항을 **단일 커밋**으로 생성합니다:
   ```
   feat({slug}): {작업 제목 요약}

   - Behaviors: {구현된 behavior 수}개
   - TDD Compliance: PASS
   - Verifier: Safe to Merge
   ```
2. 사용자에게 **다음 단계를 선택**하도록 합니다:
   - **A. 메인 브랜치에 merge**: 워크트리의 커밋을 메인 브랜치에 merge
   - **B. PR 생성**: `vibeflow/{slug}` 브랜치로 PR 생성
   - **C. 보류**: 워크트리를 유지한 채 나중에 처리
3. 선택에 따라 진행하고, merge/PR 완료 후 워크트리를 정리합니다.

### 파이프라인 중단 및 롤백

파이프라인이 **실패하거나 사용자가 중단을 요청**하면:

1. `docs/vibe/{slug}/` 디렉토리의 문서(`{slug}-spec.md`, `{slug}-plan.md`, `{slug}-lessons.md`)를 **메인 브랜치에 저장**합니다.
   - 이 문서들은 코드가 아닌 **지식 체크포인트**이므로 메인에 남겨도 안전합니다.
2. 워크트리를 **폐기**합니다:
   ```bash
   REPO_ROOT=$(git rev-parse --show-toplevel)
   git worktree remove "$WORKTREE_PATH" --force
   git branch -D "vibeflow/${slug}"
   ```
   코드 변경은 모두 사라집니다.
3. 사용자에게 보고합니다:
   ```
   [파이프라인 중단]
   - 코드 변경: 롤백 완료 (워크트리 폐기)
   - 문서 보존: docs/vibe/{slug}/ 에 저장됨
   - 재시작: /vibeflow 재호출 시 기존 문서를 감지하여 "이어하기" 가능
   ```

**재시작 시**: vibeflow를 다시 호출하면 기존 slug 디렉토리를 감지하고, SPEC + PLAN + LESSONS를 읽어 교훈을 반영한 재진행이 가능합니다.

## 운영 원칙

- 모든 의사결정과 진행 상황은 `docs/vibe/{slug}/` 디렉토리에 기록물(.md)로 남깁니다.
- builder를 **한 번만 호출하여 전체를 처리하게 하는 것은 금지**입니다.
- 각 behavior의 TDD 증거가 확보되지 않으면 다음으로 넘어가지 마세요.
- **기술 스택**: Phase 0에서 감지된 프로젝트 빌드 환경을 따릅니다. 프로젝트의 CLAUDE.md 또는 .claude/rules/에 기술 스택이 명시되어 있으면 그것을 우선합니다.

## Gotchas

1. **builder를 한 번만 호출하여 전체를 처리하려는 경향**: 반드시 behavior 1개 단위로 반복 호출해야 합니다. 한 번에 전체를 처리하면 TDD 사이클이 무효화됩니다.
2. **{slug}-plan.md 필드 누락을 확인하지 않고 Phase 1.5로 넘어가는 실수**: Phase 1 완료 후 반드시 필드 완전성 검증을 수행하세요. depends_on, testability, mock 전략, size 판정, 기존 테스트가 모두 채워져야 합니다.
3. **기존 Plan이 존재할 때 사용자 확인 없이 덮어쓰는 실수**: 동일 slug 디렉토리가 존재하면 반드시 사용자에게 "이어하기"와 "새로 만들기" 중 선택을 요청하세요.
4. **builder 중간 보고의 TDD 증거를 검증하지 않고 다음 behavior로 넘어가는 실수**: Red 실패 출력, Green 통과 출력, Refactor 통과 출력이 모두 있어야 다음 behavior로 진행할 수 있습니다.
5. **Revision 시 실패하지 않은 behavior까지 재호출하는 실수**: 실패한 behavior만 builder에 재호출하세요. 성공한 behavior를 다시 건드리면 오히려 깨질 수 있습니다.
6. **3회 revision 제한을 무시하고 계속 시도하는 실수**: 3회 후에도 미해결이면 수동 검토로 넘기세요. 무한 루프는 시간만 낭비합니다.
7. **파이프라인 중단 시 문서를 저장하지 않고 워크트리를 폐기하는 실수**: 코드는 사라져도 됩니다. 하지만 SPEC, PLAN, LESSONS 문서는 메인 브랜치에 반드시 저장한 후 폐기하세요.
8. **워크트리 없이 메인 브랜치에서 직접 작업하는 실수**: 파이프라인은 반드시 격리된 워크트리에서 실행하세요. 메인 브랜치 직접 변경은 롤백이 불가능합니다.
9. **워크트리를 레포지토리 상위 또는 무관한 경로에 생성하는 실수**: 워크트리는 반드시 `$(dirname "$REPO_ROOT")/.vibeflow-worktrees/${REPO_NAME}/${slug}` 경로에 생성하세요. `/tmp/`, 홈 디렉토리 루트, 레포보다 상위 디렉토리 등에 생성하면 안 됩니다.

