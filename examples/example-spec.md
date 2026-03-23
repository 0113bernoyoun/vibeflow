# Spec: 상품 리뷰 등록 기능

## Meta
- domain: commerce
- scope: standard
- status: approved
- created: 2026-03-15

## Overview
사용자가 구매한 상품에 대해 리뷰를 작성하고 등록할 수 있는 기능.
구매 확정된 주문에 한해 리뷰 작성이 가능하며, 별점(1~5)과 텍스트 내용을 포함한다.

## Scope
### In Scope
- 리뷰 등록 API
- 별점 및 텍스트 입력 검증
- 주문 상태 확인 (구매 확정 여부)
- 중복 리뷰 방지

### Out of Scope
- 리뷰 수정/삭제
- 리뷰 목록 조회
- 이미지 첨부

### Deferred
- 리뷰 수정/삭제 (v2)
- 판매자 답변 기능 (v2)

## Requirements

### REQ-001: 리뷰 등록
- **statement**: WHEN 사용자가 리뷰 등록을 요청하면 THE SYSTEM SHALL 별점과 내용을 저장하고 리뷰 ID를 반환한다
- **priority**: must
- **acceptance**: 리뷰 등록 후 DB에 저장되고, 응답에 리뷰 ID가 포함됨
- **related**: 없음

### REQ-002: 구매 확정 검증
- **statement**: WHEN 리뷰 등록 요청 시 THE SYSTEM SHALL 해당 주문이 구매 확정 상태인지 확인한다
- **priority**: must
- **acceptance**: 구매 확정이 아닌 주문에 대해 리뷰 등록 시 에러 반환
- **related**: [REQ-001]

### REQ-003: 중복 리뷰 방지
- **statement**: IF 동일 주문에 대해 이미 리뷰가 존재하면 THEN THE SYSTEM SHALL DuplicateReviewException을 throw한다
- **priority**: must
- **acceptance**: 같은 orderId로 두 번 등록 시 두 번째 요청이 거부됨
- **related**: [REQ-001]

### REQ-004: 별점 범위 검증
- **statement**: IF 별점이 1 미만이거나 5 초과이면 THEN THE SYSTEM SHALL InvalidRatingException을 throw한다
- **priority**: must
- **acceptance**: rating=0 또는 rating=6으로 요청 시 에러 반환
- **related**: [REQ-001]

### REQ-005: 리뷰 내용 길이 검증
- **statement**: IF 리뷰 내용이 10자 미만이거나 1000자 초과이면 THEN THE SYSTEM SHALL InvalidContentLengthException을 throw한다
- **priority**: should
- **acceptance**: 9자 내용으로 등록 시 에러 반환, 1001자 내용으로 등록 시 에러 반환
- **related**: [REQ-001]

## Business Rules

### BR-001: 리뷰 작성 자격
- **조건**: 주문 상태가 ORDER_CONFIRMED이고, 주문자 본인일 때
- **결과**: 리뷰 작성이 허용됨

### BR-002: 리뷰당 1개 제한
- **조건**: 하나의 주문에 대해
- **결과**: 최대 1개의 리뷰만 작성 가능

## Edge Cases

### EC-001: 주문이 존재하지 않음
- **상황**: 존재하지 않는 orderId로 리뷰 등록 요청
- **기대 동작**: OrderNotFoundException throw
- **related**: [REQ-002]

### EC-002: 다른 사용자의 주문
- **상황**: 본인의 주문이 아닌 orderId로 리뷰 등록 요청
- **기대 동작**: UnauthorizedReviewException throw
- **related**: [REQ-002]

### EC-003: 빈 문자열 리뷰
- **상황**: 내용이 빈 문자열("")이거나 공백만 있는 경우
- **기대 동작**: InvalidContentLengthException throw (trim 후 길이 검증)
- **related**: [REQ-005]

## Assumptions

| ID | 가정 | 확인 상태 | 확인 방법 |
|---|---|---|---|
| ASM-001 | Order Entity에 status 필드가 존재한다 | confirmed | 코드 확인 |
| ASM-002 | 인증된 사용자의 memberId가 컨텍스트에서 추출 가능하다 | confirmed | 사용자 확인 |
| ASM-003 | Review Entity는 신규 생성이 필요하다 | confirmed | 코드 확인 (기존 없음) |

## Open Questions

| ID | 질문 | 영향 범위 | 차단 여부 |
|---|---|---|---|
| OQ-001 | 리뷰 등록 시 포인트 적립이 필요한가? | REQ-001 확장 | non-blocking |

## API Specification

### POST /api/reviews

**Request:**
```json
{
  "orderId": "uuid",
  "rating": 5,
  "content": "좋은 상품입니다. 추천합니다."
}
```

**Response (201):**
```json
{
  "reviewId": "uuid",
  "orderId": "uuid",
  "rating": 5,
  "content": "좋은 상품입니다. 추천합니다.",
  "createdAt": "2026-03-15T10:30:00"
}
```

**Error Responses:**
- 400: InvalidRatingException, InvalidContentLengthException
- 403: UnauthorizedReviewException
- 404: OrderNotFoundException
- 409: DuplicateReviewException

## Data Model

### Review (신규)

| 필드 | 타입 | 설명 |
|---|---|---|
| id | UUID (v7) | PK |
| orderId | UUID | FK → Order |
| memberId | UUID | FK → Member |
| rating | Integer | 1~5 |
| content | String (longtext) | 리뷰 내용 |
| createdAt | LocalDateTime | 작성일시 |

## Codebase Context

- 유사 기능: ProductQna (상품 Q&A) → 등급: **참고**
  - 통과: 컨벤션 준수, 완결성
  - 미통과: 테스트 부재, 엣지케이스 미처리
  - 권장: 구조만 참고, 검증 로직은 새로 작성
- 재사용 적합: CoreOrderService.getOrder()
- 영향 범위: 신규 생성 (기존 코드 변경 없음)
