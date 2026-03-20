# Verification Report 템플릿

검증 완료 후 아래 양식으로 보고합니다:

```markdown
## Verification Report

### TDD Compliance
| Behavior | Red 증거 | Green 증거 | Refactor 증거 | 판정 |
|---|---|---|---|---|
| Behavior 1: {이름} | {있음/없음} | {있음/없음} | {있음/없음} | PASS/FAIL |
| Behavior 2: {이름} | ... | ... | ... | ... |

**TDD 준수율**: {통과 behavior 수}/{전체 behavior 수}

### Existing Test Management
| 테스트 파일 | 방침 | Baseline 수 | 현재 수 | 무단 변경 | 판정 |
|---|---|---|---|---|---|
| {파일명} | 보호/보강/수정/삭제 | {N}개 | {N}개 | {없음/있음} | PASS/FAIL |

**기존 테스트 관리 적합율**: {통과}/{전체}
- 보호 위반: {있으면 명시}
- 근거 없는 변경: {있으면 명시}
- 대체 없는 삭제: {있으면 명시}

### Spec-Code Alignment
| Behavior | Input 일치 | Output 일치 | Constraints 일치 | 테스트 커버 | 판정 |
|---|---|---|---|---|---|
| Behavior 1: {이름} | OK/MISMATCH | OK/MISMATCH | OK/MISMATCH | OK/PARTIAL/MISSING | PASS/FAIL |

- PLAN.md behavior 전수 대조: PASS/FAIL
- 누락 항목: {있으면 명시}
- 불일치 항목: {있으면 명시}

### Test Quality
- 전체 테스트 실행 결과: {BUILD SUCCESSFUL/FAILED}
- 의미 없는 테스트: {있으면 명시}

### Clean-up Activity
{직접 수행한 리팩토링 및 정리 내역}

### Build Verification
- 컴파일 (`./gradlew compileJava`): SUCCESS/FAIL
- 빌드 (`./gradlew build`): SUCCESS/FAIL
- 경고 사항: {있으면 명시}

### Final Verdict
**"Safe to Merge"** 또는 **"Requires Revision"**
- TDD 미준수 behavior가 1개라도 있으면: Requires Revision
- 빌드 또는 컴파일이 실패하면: Requires Revision
- 스펙-코드 불일치 behavior가 1개라도 있으면: Requires Revision
- 보호 대상 테스트가 변경되었으면: Requires Revision
- PLAN.md에 없는 기존 테스트 변경이 발견되면: Requires Revision
- 대체 테스트 없이 기존 테스트가 삭제되었으면: Requires Revision
- 사유: {수정이 필요하면 이유 명시}
```
