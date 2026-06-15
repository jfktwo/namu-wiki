# Matches And Inquiries

## 목적

매칭은 수요와 재고 조건이 맞는 후보를 보여주는 기능이고, 문의는 특정 매칭에 대해 상대방에게 거래 의사를 보내는 기능입니다.

## 구성 파일

| 파일 | 역할 |
|---|---|
| `src/features/matches/api.ts` | 매칭 조회 API |
| `src/features/inquiries/api.ts` | 문의 API |
| `src/features/inquiries/model.ts` | 문의 상태 계산 helper |
| `src/pages/MatchListPage.tsx` | 매칭 목록 |
| `src/pages/MatchDetailPage.tsx` | 매칭 상세 |
| `src/pages/InquiryListPage.tsx` | 문의 목록/상태 변경 |

## 매칭 API

- `listMatches(maxKm?)`: `GET /api/matches?max_km=...`
- `getMatch(id)`: `GET /api/matches/{id}`

매칭 데이터에는 `demand_id`, `stock_id`, `distance_km`, `total_price`가 있고, 조회 방식에 따라 `demands` 또는 `stocks` 객체가 포함됩니다.

## 문의 API

- `listInquiries()`: `GET /api/inquiries`
- `createInquiry(match_id, receiver_id)`: `POST /api/inquiries`
- `updateInquiryStatus(id, status)`: `PATCH /api/inquiries/{id}/status?status=...`

문의 상태는 `requested`, `accepted`, `rejected`, `cancelled`입니다.

## 매칭 목록 흐름

1. 거리 필터 기본값은 50km입니다.
2. `listMatches(maxKm)`와 `listInquiries()`를 함께 호출합니다.
3. 이미 보낸 문의의 `match_id`를 Set으로 만들어 문의 버튼 상태에 반영합니다.
4. 카드를 누르면 매칭 상세로 이동합니다.
5. 문의 버튼을 누르면 `createInquiry()`를 호출합니다.

## 매칭 상세 흐름

상세 화면은 매칭 요약, 재고 정보, 수요 정보를 역할에 따라 순서를 달리 보여줍니다. 구매자는 재고를 먼저 보고, 공급자는 수요를 먼저 봅니다.

이미 문의한 매칭이면 버튼은 비활성화되고 `문의 완료` 상태가 됩니다.

## 문의 목록 흐름

문의 화면은 `보낸 문의`와 `받은 문의` 탭을 나눕니다.

- 받은 문의가 `requested`이면 수락/거절 가능
- 보낸 문의가 `requested`이면 취소 가능
- 상태 변경 후 목록을 다시 불러옵니다.

## 리뷰 메모

- 최근 수정으로 이미 보낸 문의가 매칭 목록/상세에 반영됩니다.
- `receiverId`는 `match.stocks?.user_id ?? match.demands?.user_id`로 계산합니다. 역할별 수신자가 올바른지 백엔드에서도 검증하는 방어가 있으면 더 안전합니다.
- 문의에는 메시지 본문이 없습니다. 현재는 상태 요청 모델이라 대화 기능으로 확장하려면 messages 테이블 또는 inquiry message 컬럼이 필요합니다.
- catch-all alert가 많습니다. 서버 에러 detail을 표시하면 운영 중 원인 파악이 쉬워집니다.

## 거리 계산 로직

매칭 카드와 상세 화면에 표시되는 `몇 km` 값은 백엔드 `matches.distance_km` 필드입니다. 이 값은 수요 주소의 좌표와 재고 주소의 좌표 사이를 계산한 거리입니다.

계산 위치는 `backend/app/domains/matches/service.py`입니다.

```python
dist = _haversine_km(demand["lat"], demand["lng"], stock["lat"], stock["lng"])
```

`_haversine_km()` 함수는 위도/경도를 라디안으로 바꾼 뒤 haversine 공식으로 지구 곡률을 반영한 직선거리를 km 단위로 계산합니다. 반환값은 매칭 row에 `round(dist, 2)`로 저장됩니다.

```python
{
    "demand_id": demand["id"],
    "stock_id": stock["id"],
    "distance_km": round(dist, 2),
    "total_price": total_price,
}
```

중요한 점은 이 거리가 도로 주행거리나 배송거리 기준이 아니라 지도 좌표 간 직선거리 기준이라는 점입니다. 주소가 바뀌거나 위치 선택 좌표가 바뀌면 매칭 거리도 달라집니다.

매칭 생성 조건은 다음 순서로 적용됩니다.

1. 수요와 재고의 `species`가 같아야 합니다.
2. 수요와 재고의 `spec`이 같아야 합니다.
3. 재고 수량이 수요 수량 이상이어야 합니다.
4. `재고 단가 * 수요 수량`이 수요 예산 이하이어야 합니다.
5. 수요 좌표와 재고 좌표의 거리를 계산합니다.
6. 계산된 거리가 300km 이하면 매칭 후보가 됩니다.

프론트의 거리 필터 `50km`, `100km`, `200km`, `전국`은 이미 계산되어 저장된 `distance_km` 값에 대해 `/api/matches?max_km=...` 파라미터로 추가 필터링하는 방식입니다.
