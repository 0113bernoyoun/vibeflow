# Builder 완료 보고 형식

각 behavior 완료 시 아래 형식으로 보고합니다:

```markdown
## Behavior {N}: {이름} - 완료

### Spec Reading
- Behavior: {N}. {이름}
- Input: ...
- Output: ...
- Constraints: ...

### Baseline & Assessment
- 기존 테스트 수: {N}개
- 보호 대상: {M}개 → 전체 통과
- 보강: {내역 또는 "해당 없음"}
- 수정: {내역 또는 "해당 없음"}
- 삭제: {내역 및 사유 또는 "해당 없음"}

### Red Phase
- 테스트명: ...
- 실패 메시지: ...

### Green Phase
- 변경 파일: ...
- 전체 테스트 결과: PASS (N개)
- 기존 테스트 보호 대상: PASS ({M}개 전체 유지)

### Refactor Phase
- 수행 내역: ...
- 전체 테스트 결과: PASS (N개)
- 기존 테스트 보호 대상: PASS ({M}개 전체 유지)
```
