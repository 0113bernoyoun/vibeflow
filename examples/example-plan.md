# PLAN: 상품 리뷰 등록 기능

## Context
구매 확정된 주문에 대해 사용자가 별점과 텍스트 리뷰를 등록하는 기능.
신규 Review Entity 생성 및 등록 API 구현.

## Assumptions

| 전제 | 내용 | 확인 방법 |
|------|------|-----------|
| Order Entity 존재 | status 필드 포함, ORDER_CONFIRMED 상태 사용 가능 | 코드 확인 |
| 인증 컨텍스트 | memberId가 SecurityContext에서 추출 가능 | 사용자 확인 |
| Review Entity 없음 | 신규 생성 필요 | 코드 확인 |

## Architecture
- **신규**: Review Entity, ReviewRepository, SaveReviewService, ReviewController
- **기존 재사용**: CoreOrderService.getOrder()
- **참고만 (경고 등급 아님)**: ProductQna 패키지 구조 참고, 검증 로직은 새로 작성

## Existing Test Baseline

| 테스트 파일 | 테스트 메서드 수 | 실행 결과 | 품질 등급 | 처리 방침 | 사유 |
|---|---|---|---|---|---|
| CoreOrderServiceTest.java | 5개 | PASS | 양호 | 보호 | 기존 주문 조회 테스트, 변경 불필요 |

**기존 테스트 총 수**: 5개
**보호 대상**: 5개
**보강/수정 대상**: 0개

## Behaviors (순서대로)

### Behavior 1: Review Entity 생성
- spec-ref: REQ-001
- 설명: Review Entity와 JPA Repository를 생성한다
- 입력: Review Entity 클래스 정의
- 기대 결과: Entity가 정상적으로 영속화되고 ID가 생성됨
- 예외 상황: 없음 (Entity 생성 단계)
- 영향 범위: Review.java, ReviewRepository.java
- depends_on: 없음
- testability: Review를 save한 후 findById로 조회하여 필드값 일치 확인
- mock 전략: 없음 (JPA 테스트)
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

### Behavior 2: 별점 범위 검증
- spec-ref: REQ-004
- 설명: 별점이 1~5 범위 밖이면 InvalidRatingException을 throw한다
- 입력: rating = 0 또는 rating = 6
- 기대 결과: InvalidRatingException 발생
- 예외 상황: rating이 null인 경우도 예외 처리
- 영향 범위: SaveReviewService.java, ReviewBaseCode.java, InvalidRatingException.java
- depends_on: [1] Review Entity 존재
- testability: assertThatThrownBy로 예외 타입과 메시지 확인
- mock 전략: 없음 (순수 검증 로직)
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

### Behavior 3: 리뷰 내용 길이 검증
- spec-ref: REQ-005
- 설명: 리뷰 내용이 10자 미만 또는 1000자 초과이면 InvalidContentLengthException을 throw한다
- 입력: content = "짧음" (4자) 또는 1001자 문자열
- 기대 결과: InvalidContentLengthException 발생
- 예외 상황: 공백만 있는 문자열 (trim 후 검증)
- 영향 범위: SaveReviewService.java, InvalidContentLengthException.java
- depends_on: [1] Review Entity 존재
- testability: assertThatThrownBy로 예외 타입 확인, trim 후 길이 검증 확인
- mock 전략: 없음 (순수 검증 로직)
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

### Behavior 4: 구매 확정 상태 검증
- spec-ref: REQ-002
- 설명: 주문이 구매 확정(ORDER_CONFIRMED) 상태가 아니면 예외를 throw한다
- 입력: orderId가 가리키는 Order의 status가 PENDING
- 기대 결과: OrderNotConfirmedException 발생
- 예외 상황: Order가 존재하지 않는 경우 (EC-001) → OrderNotFoundException
- 영향 범위: SaveReviewService.java, OrderNotConfirmedException.java
- depends_on: [1] Review Entity 존재
- testability: assertThatThrownBy로 예외 타입 확인
- mock 전략: CoreOrderService를 Mock하여 Order 상태를 제어
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

### Behavior 5: 중복 리뷰 방지
- spec-ref: REQ-003
- 설명: 동일 orderId로 이미 리뷰가 존재하면 DuplicateReviewException을 throw한다
- 입력: 이미 리뷰가 존재하는 orderId
- 기대 결과: DuplicateReviewException 발생
- 예외 상황: 없음
- 영향 범위: SaveReviewService.java, ReviewRepository.java, DuplicateReviewException.java
- depends_on: [1] Review Entity 존재, [4] 구매 확정 검증 통과 후 실행
- testability: 리뷰를 하나 저장한 후 같은 orderId로 재등록 시 예외 확인
- mock 전략: CoreOrderService를 Mock
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

### Behavior 6: 리뷰 정상 등록 (Happy Path)
- spec-ref: REQ-001
- 설명: 모든 검증을 통과한 리뷰를 저장하고 리뷰 ID를 반환한다
- 입력: 유효한 orderId, rating=5, content="좋은 상품입니다. 추천합니다."
- 기대 결과: Review가 DB에 저장되고, 리뷰 ID가 반환됨
- 예외 상황: 없음 (모든 예외는 이전 behavior에서 처리)
- 영향 범위: SaveReviewService.java, ReviewController.java, SaveReviewRequest.java, DetailReviewResponse.java
- depends_on: [2][3][4][5] 모든 검증 behavior
- testability: save 후 반환된 ID로 조회하여 필드값 일치 확인
- mock 전략: CoreOrderService를 Mock하여 ORDER_CONFIRMED 반환
- size 판정: 단일 입력-결과 대응 ✓
- 기존 테스트: 없음

## Logical Harness

- [x] Behavior 1의 Entity가 Behavior 2~6의 전제 조건으로 작동하는가?
- [x] Behavior 2~5의 검증이 Behavior 6의 Happy Path 전에 모두 수행되는가?
- [x] 모든 예외 상황(EC-001, EC-002, EC-003)에 대한 behavior가 포함되어 있는가?
- [x] depends_on이 순서와 일치하는가?
- [x] Assumptions의 전제가 모든 behavior에 일관되게 적용되었는가?
