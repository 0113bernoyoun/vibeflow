# VibeFlow

TDD 기반 설계-구현-검증 파이프라인. 아이디어를 behavior 단위 루프로 분해하고, Red-Green-Refactor 사이클을 강제하여 프로덕션 품질의 코드를 생성합니다.

## Why VibeFlow?

AI 코드 생성의 가장 큰 문제는 **"동작하는 것 같은 코드"**를 만들어내는 것입니다. 테스트 없이, 검증 없이, 기존 코드와의 정합성 확인 없이.

VibeFlow는 이 문제를 **프로세스로 해결**합니다:

1. **스펙부터 시작** — 코드를 쓰기 전에 "무엇을 왜" 정의
2. **Behavior 단위 분해** — 거대한 기능을 테스트 가능한 단위로 쪼갬
3. **TDD 강제** — 각 behavior마다 Red → Green → Refactor 사이클 실행 + 증거 수집
4. **독립 검증** — 구현자(builder)와 검증자(verifier)를 분리하여 자기 검증 함정 방지

## Pipeline

```
[spec-writer] → spec.md
                    ↓
[vibeflow] → [planner] → plan.md → [builder] × N → [verifier]
                                     ↑     ↓           ↓
                                     └─ behavior ─┘   Safe to Merge
                                        단위 루프      or Revision
```

| Phase | Skill | 역할 |
|---|---|---|
| Spec | `spec-writer` | Critic 인터뷰 + 도메인 리서치 + EARS 표기법 요구사항 |
| Plan | `planner` | Behavior 분해 + 의존성 순서 + 테스트 전략 |
| Build | `builder` | Behavior 1개씩 Red-Green-Refactor + 자기 비판 |
| Verify | `verifier` | TDD 준수 검증 + 스펙-코드 대조 + 빌드 확인 |
| Orchestrate | `vibeflow` | 전체 루프 제어 + 워크트리 격리 + Revision 관리 |

## Quick Start

### 설치

```bash
# GitHub 마켓플레이스로 설치
/plugin marketplace add 0113bernoyoun/vibeflow
/plugin install vibeflow
```

또는 로컬 설치:

```bash
# 레포 클론
git clone https://github.com/0113bernoyoun/vibeflow.git

# 프로젝트의 .claude/skills/ 에 복사
cp -r vibeflow/skills/* your-project/.claude/skills/
```

### 사용법

#### 전체 파이프라인 (TDD로 기능 구현)

```
/vibeflow 장바구니에 쿠폰 적용 기능을 추가해줘
```

VibeFlow가 자동으로:
1. 스펙 문서 필요 여부를 물어봅니다
2. Behavior를 설계합니다
3. 각 behavior를 TDD로 구현합니다
4. 최종 검증 후 커밋합니다

#### 스펙만 작성 (독립 사용)

```
/spec-writer 상품 리뷰 기능의 스펙을 작성해줘
```

Spec-writer가 Critic 인터뷰로 요구사항을 정제하고, REQ-ID + EARS 표기법의 추적 가능한 스펙 문서를 생성합니다. 이 스펙은 나중에 vibeflow에 입력으로 사용할 수 있습니다.

#### 개별 스킬 직접 호출

```
/planner plan-path=docs/vibe/my-feature/my-feature-plan.md
/builder plan-path=docs/vibe/my-feature/my-feature-plan.md behavior=1
/verifier plan-path=docs/vibe/my-feature/my-feature-plan.md
```

## Features

### Spec Writer
- **Critic 인터뷰**: 모호함을 불허하는 구조화된 질문
- **EARS 표기법**: 5가지 패턴(Ubiquitous, Event-driven, State-driven, Unwanted, Optional)으로 테스트 가능한 요구사항 작성
- **REQ-ID 추적성**: 모든 요구사항에 고유 ID 부여 → planner의 behavior까지 추적
- **기존 코드 품질 평가**: 유사 기능 발견 시 신뢰/참고/경고 3등급 분류
- **Figma 연동**: Figma MCP가 있으면 디자인에서 직접 스펙 추출 (선택적)

### Planner
- **Mode A/B**: spec-writer의 스펙이 있으면 검증/보강 후 behavior 설계, 없으면 자체 생성
- **Behavior 크기 기준**: "단일 입력-결과 대응"으로 TDD 사이클에 적합한 크기 강제
- **Testability 검증**: assert 불가능한 behavior 사전 차단
- **기존 테스트 분석**: 보호/보강/수정/삭제 등급으로 기존 테스트 관리 방침 결정

### Builder
- **Critic 자기 점검**: Red/Green/Refactor 각 단계 후 자기 비판
- **실행 증거 필수**: "통과할 것이다"는 추측 금지, 실행 출력만 인정
- **Spec 문제 보고**: `[Ambiguous Spec]`, `[Spec Conflict]`, `[Missing Behavior]` 태그로 파이프라인에 즉시 보고
- **2회 재시도 제한**: 자체 수정은 최대 2회, 이후 중단 보고

### Verifier
- **Builder 불신**: builder 보고서를 맹신하지 않고 직접 테스트 실행
- **TDD 준수 검증**: Red 증거 → Green 증거 → Refactor 증거 전수 확인
- **스펙-코드 1:1 대조**: Input/Output/Constraints 불일치 탐지
- **무단 테스트 변경 탐지**: `@Disabled` 추가, assert 약화 등 적발

### Orchestrator (vibeflow)
- **Git 워크트리 격리**: 파이프라인 전체를 격리된 브랜치에서 실행
- **빌드 도구 자동 감지**: Gradle, Maven, npm 등 자동 인식
- **Revision 루프**: verifier 실패 시 최대 3회 자동 수정 시도
- **롤백 안전망**: 실패 시 코드 롤백 + 문서(spec/plan/lessons) 보존

## Supported Tech Stacks

| 빌드 도구 | 감지 기준 | 테스트 명령 |
|---|---|---|
| Gradle | `gradlew` 또는 `build.gradle` | `./gradlew test` |
| Maven | `pom.xml` | `mvn test` |
| npm | `package.json` | `npm test` |
| Make | `Makefile` | `make test` |

프로젝트의 `CLAUDE.md`에 기술 스택이 명시되어 있으면 그것을 우선합니다.

VibeFlow의 스킬 내부는 `{test_all}`, `{compile}` 등 플레이스홀더를 사용하므로 **특정 기술 스택에 종속되지 않습니다**. `examples/`의 산출물은 Spring Boot/Java 기준이지만, 실제 사용 시 프로젝트의 빌드 도구에 맞게 자동 치환됩니다.

## Output Artifacts

VibeFlow가 생성하는 산출물은 `docs/vibe/{slug}/` 디렉토리에 저장됩니다:

| 파일 | 생성 시점 | 내용 |
|---|---|---|
| `{slug}-spec.md` | Spec/Plan Phase | 요구사항 (REQ-ID, EARS, 비즈니스 규칙, 엣지케이스) |
| `{slug}-plan.md` | Plan Phase | Behavior 명세 (입력/출력/의존성/테스트전략) |
| `{slug}-lessons.md` | Verify Phase | 교훈 (도메인/프로세스/기술적 발견) |

예시 산출물은 `examples/` 디렉토리를 참고하세요.

## Skill Structure

```
vibeflow/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── vibeflow/
│   │   └── SKILL.md              # Orchestrator (431 lines)
│   ├── spec-writer/
│   │   ├── SKILL.md              # Spec designer (415 lines)
│   │   └── references/
│   │       ├── output-contract.md
│   │       ├── ears-guide.md
│   │       └── furps-checklist.md
│   ├── planner/
│   │   ├── SKILL.md              # Behavior planner (347 lines)
│   │   └── references/
│   │       ├── plan-template.md
│   │       └── domains/
│   │           └── _template.md
│   ├── builder/
│   │   ├── SKILL.md              # TDD engine (222 lines)
│   │   └── references/
│   │       └── completion-report-template.md
│   └── verifier/
│       ├── SKILL.md              # Quality guardian (174 lines)
│       └── references/
│           └── verification-report-template.md
├── examples/
│   └── (example spec, plan, and verification report)
├── skill-verifier.md
├── README.md
└── LICENSE
```

## Agent Skills Compatibility

VibeFlow의 SKILL.md 파일은 [Agent Skills 오픈 표준](https://agentskills.io)을 준수합니다. Claude Code 외에 이 표준을 지원하는 도구(Codex, Gemini CLI, Cursor 등)에서도 사용할 수 있습니다.

## License

Apache License 2.0 — See [LICENSE](LICENSE).
