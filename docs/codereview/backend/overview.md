# Backend Overview

## 목적

이 문서는 프론트엔드가 호출하는 백엔드 API를 이해하기 위한 연결 리뷰입니다. 백엔드는 FastAPI + Supabase 클라이언트 구조이며, 도메인별로 router, service, repository가 나뉘어 있습니다.

## 전체 구조

| 영역 | 파일/폴더 | 역할 |
|---|---|---|
| 앱 진입 | `backend/app/main.py` | FastAPI 앱 생성, CORS, router 등록 |
| 설정 | `backend/app/core/config.py` | 환경 변수 |
| DB | `backend/app/core/database.py` | Supabase client 생성 |
| 인증 | `backend/app/core/security.py`, `deps.py` | JWT 생성/검증, 역할 guard |
| 에러 | `backend/app/core/errors.py` | HTTPException helper |
| 도메인 | `backend/app/domains/*` | users, demands, stocks, matches, inquiries, moderation |
| DB 스키마 | `database/migrations/*.sql` | Supabase 테이블 정의 |

## API 경계

| 프론트 기능 | 백엔드 경로 | 설명 |
|---|---|---|
| 인증 | `/api/auth/*` | 회원가입, 로그인, 내 정보 |
| 수요 | `/api/demands` | 수요 CRUD |
| 재고 | `/api/stocks` | 재고 CRUD |
| 이미지 | `/api/stocks/upload` | 재고 사진 업로드 |
| 매칭 | `/api/matches` | 조건 기반 매칭 조회 |
| 문의 | `/api/inquiries` | 문의 요청과 상태 변경 |
| 관리자 | `/api/admin/*` | 사용자 관리, 숨김 처리 |

## 인증 방식

로그인/회원가입 성공 시 JWT를 발급합니다. JWT payload에는 `sub`와 `role`이 들어갑니다. 프론트는 이 토큰을 `Authorization: Bearer ...` 헤더로 보냅니다.

백엔드의 `get_current_user`는 토큰을 decode해서 현재 사용자 payload를 반환합니다. `require_role`은 특정 역할만 접근하도록 막습니다.

## 도메인 흐름

### 수요

구매자만 생성할 수 있습니다. 생성/수정 후 `matches.service.recalculate_for_demand`를 호출해 매칭 후보를 다시 계산합니다.

### 재고

공급자만 생성할 수 있습니다. 생성/수정 후 `recalculate_for_stock`을 호출합니다. 사진 업로드는 `stock-photos` 버킷에 저장하고 public URL을 반환합니다.

### 매칭

수종, 규격, 수량, 예산, 거리 조건을 기준으로 수요와 재고를 매칭합니다. 거리는 haversine 공식으로 계산합니다. 내부 계산 한도는 300km이고, 프론트 목록에서는 `max_km` 필터를 추가로 보냅니다.

### 문의

문의 생성 시 `match_id + requester_id` 중복을 막습니다. 목록은 보낸 문의와 받은 문의로 나누어 반환합니다.

### 관리자

사용자 목록, 역할 변경, 사용자 삭제, 수요/재고 숨김 처리를 제공합니다. moderation은 별도 service 파일 없이 router에서 직접 Supabase를 호출합니다.

## 프론트와 맞물리는 주의점

- CORS는 현재 `http://localhost:5173`만 허용합니다. Vercel 또는 모바일 앱 배포 시 허용 origin을 추가해야 합니다.
- 이미지 업로드는 공급자 권한만 허용합니다. 수요자 이미지 업로드가 필요하면 별도 엔드포인트 또는 권한 정책이 필요합니다.
- 백엔드 메시지 문자열도 한글 인코딩이 깨진 흔적이 있습니다.
- Supabase secret key를 백엔드에서 사용하므로 백엔드는 반드시 서버 환경에서만 실행해야 합니다.
- 문의 생성 시 프론트가 보내는 `receiver_id`가 실제 매칭 상대인지 백엔드에서 검증하지 않습니다. 데이터 무결성을 위해 검증을 추가하는 편이 좋습니다.

## 매칭 거리 계산 상세

매칭의 `distance_km`는 `backend/app/domains/matches/service.py`에서 계산합니다. 수요의 `lat`, `lng`와 재고의 `lat`, `lng`를 `_haversine_km()` 함수에 넣어 km 단위 거리를 구합니다.

```python
dist = _haversine_km(demand["lat"], demand["lng"], stock["lat"], stock["lng"])
```

`_haversine_km()`는 지구 반지름 `6371.0km`를 기준으로 haversine 공식을 적용합니다. 그래서 단순 평면거리가 아니라 위도/경도 기반의 지구 곡률 반영 직선거리입니다.

매칭 후보가 만들어지는 조건은 `_match_pair()`에 들어 있습니다.

1. `demand["species"] == stock["species"]`
2. `demand["spec"] == stock["spec"]`
3. `stock["quantity"] >= demand["quantity"]`
4. `stock["unit_price"] * demand["quantity"] <= demand["budget"]`
5. `distance_km <= 300`

조건을 통과하면 다음 형태로 `matches` 테이블에 저장됩니다.

```python
{
    "demand_id": demand["id"],
    "stock_id": stock["id"],
    "distance_km": round(dist, 2),
    "total_price": total_price,
}
```

프론트에서 사용자가 보는 `50km`, `100km`, `200km` 필터는 이 저장된 `distance_km` 값을 기준으로 백엔드 `list_matches(user, max_km)`에서 다시 걸러집니다.

주의할 점은 현재 거리가 도로 주행거리, 배송거리, 시간 기준이 아니라 주소 좌표 사이의 직선거리라는 점입니다.
