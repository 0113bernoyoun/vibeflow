# EARS 표기법 가이드

Easy Approach to Requirements Syntax의 5가지 패턴입니다.
요구사항 statement 작성 시 가장 적합한 패턴을 선택합니다.

## 패턴 목록

### 1. Ubiquitous (항상 성립)

**형식**: `THE SYSTEM SHALL {동작}`

**사용 시점**: 조건 없이 항상 성립해야 하는 제약

**예시**:
- THE SYSTEM SHALL 모든 API 응답에 request-id를 포함한다
- THE SYSTEM SHALL 비밀번호를 평문으로 저장하지 않는다
- THE SYSTEM SHALL 금액을 BigDecimal로 처리한다

### 2. Event-driven (이벤트 발생 시)

**형식**: `WHEN {이벤트} THE SYSTEM SHALL {동작}`

**사용 시점**: 특정 이벤트가 발생했을 때 시스템이 반응해야 할 때

**예시**:
- WHEN 사용자가 주문을 생성하면 THE SYSTEM SHALL 재고를 차감한다
- WHEN 결제가 완료되면 THE SYSTEM SHALL 주문 상태를 PAID로 변경한다
- WHEN 파일이 업로드되면 THE SYSTEM SHALL 바이러스 스캔을 실행한다

### 3. State-driven (상태 유지 중)

**형식**: `WHILE {상태} THE SYSTEM SHALL {동작}`

**사용 시점**: 특정 상태가 유지되는 동안 시스템이 해야 할 행동

**예시**:
- WHILE 주문 상태가 PENDING인 동안 THE SYSTEM SHALL 주문 취소를 허용한다
- WHILE 세션이 유효한 동안 THE SYSTEM SHALL 토큰 자동 갱신을 수행한다

### 4. Unwanted Behavior (예외/에러 처리)

**형식**: `IF {예외 조건} THEN THE SYSTEM SHALL {대응}`

**사용 시점**: 에러, 예외, 비정상 상황에 대한 처리

**예시**:
- IF 결제 금액이 0원 이하이면 THEN THE SYSTEM SHALL InvalidAmountException을 throw한다
- IF 외부 API 응답이 5초 이내에 없으면 THEN THE SYSTEM SHALL timeout 에러를 반환한다
- IF 재고가 부족하면 THEN THE SYSTEM SHALL OutOfStockException을 throw하고 주문을 생성하지 않는다

### 5. Optional Feature (선택적 기능)

**형식**: `WHERE {기능 포함 조건} THE SYSTEM SHALL {동작}`

**사용 시점**: 특정 조건/설정/구성에서만 활성화되는 기능

**예시**:
- WHERE 알림 수신에 동의한 사용자에게 THE SYSTEM SHALL 주문 상태 변경 알림을 발송한다
- WHERE 프리미엄 요금제인 경우 THE SYSTEM SHALL 고급 통계 대시보드를 표시한다

## 복합 패턴

복잡한 요구사항은 패턴을 조합할 수 있습니다:

**Event + Unwanted**:
`WHEN 사용자가 주문을 생성하면, IF 재고가 부족하면 THEN THE SYSTEM SHALL OutOfStockException을 throw한다`

**State + Event**:
`WHILE 주문 상태가 PENDING인 동안, WHEN 사용자가 취소를 요청하면 THE SYSTEM SHALL 주문 상태를 CANCELLED로 변경한다`

## 작성 시 주의사항

1. **모호한 동사 금지**: "처리한다", "관리한다", "대응한다" → 구체적 동작으로 변환
2. **측정 가능한 기준**: "빠르게 응답한다" → "P95 응답 시간 500ms 이내"
3. **하나의 statement에 하나의 동작**: "A를 하고 B를 한다" → 두 개의 REQ로 분리
4. **시스템이 주어**: 사용자의 행동이 아닌 시스템의 반응을 기술
