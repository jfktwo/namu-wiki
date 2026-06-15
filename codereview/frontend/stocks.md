# Stocks

## 목적

재고 기능은 공급자(`seller`)가 보유한 나무 재고를 등록하고, 사진과 위치를 포함해 관리하는 영역입니다.

## 구성 파일

| 파일 | 역할 |
|---|---|
| `src/features/stocks/api.ts` | 재고 CRUD API |
| `src/pages/StockListPage.tsx` | 재고 목록 |
| `src/pages/StockCreatePage.tsx` | 재고 등록 |
| `src/pages/StockDetailPage.tsx` | 재고 상세 |
| `src/pages/StockEditPage.tsx` | 재고 수정/삭제 |
| `src/shared/components/ImageUpload.tsx` | 이미지 업로드 UI |
| `src/shared/components/LocationPicker.tsx` | 주소/좌표 선택 UI |

## API

- `listStocks(status?)`: `GET /api/stocks`
- `getStock(id)`: `GET /api/stocks/{id}`
- `createStock(data)`: `POST /api/stocks`
- `updateStock(id, data)`: `PATCH /api/stocks/{id}`
- `deleteStock(id)`: `DELETE /api/stocks/{id}`
- 이미지 업로드: `POST /api/stocks/upload`

## 데이터 모델

재고는 `species`, `spec`, `quantity`, `unit_price`, `address`, `lat`, `lng`, `photo_urls`, `expires_at`, `status`를 갖습니다.

`photo_urls`는 배열이며, 목록/상세/수정 화면에서 이미지를 표시합니다. `ImageUpload`는 업로드 성공 시 URL 배열을 부모 폼에 넘깁니다.

## 화면 흐름

1. 목록은 `listStocks()`로 데이터를 가져와 2열 카드로 표시합니다.
2. 공급자만 등록 버튼을 볼 수 있습니다.
3. 등록 화면은 재고 정보, 위치, 이미지를 입력합니다.
4. 상세 화면은 대표 사진과 썸네일 갤러리를 보여줍니다.
5. 수정 화면은 기존 위치와 이미지 URL을 초기값으로 받아 편집합니다.

## 이미지 업로드

`ImageUpload`는 파일 input을 숨기고, 커스텀 버튼으로 파일 선택을 엽니다. 파일은 `multipart/form-data`로 `/api/stocks/upload`에 업로드되고, 응답의 public URL을 `photo_urls`에 추가합니다.

현재 기본 최대 개수는 5개입니다. 이전 요구사항에서는 재고당 3개 이미지를 사용했지만, UI 컴포넌트는 5개까지 허용합니다.

## 리뷰 메모

- 업로드 파일 크기 제한은 프론트에 없습니다. 백엔드 또는 프론트에서 제한을 명시하는 편이 좋습니다.
- 업로드는 순차 실행입니다. 여러 장 업로드 시 느릴 수 있지만 실패 처리 단순성은 좋습니다.
- 공급자 권한은 백엔드 `/api/stocks/upload`에서 `require_role("seller")`로 제한합니다.
- 재고 등록/수정 폼도 숫자 및 날짜 검증을 더 보강할 여지가 있습니다.

