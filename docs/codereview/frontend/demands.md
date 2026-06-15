# Demands

## 목적

수요 기능은 구매자(`buyer`)가 필요한 나무 조건을 등록하고, 등록된 수요를 목록/상세/수정 화면에서 관리하는 영역입니다.

## 구성 파일

| 파일 | 역할 |
|---|---|
| `src/features/demands/api.ts` | 수요 CRUD API |
| `src/pages/DemandListPage.tsx` | 수요 목록 |
| `src/pages/DemandCreatePage.tsx` | 수요 등록 |
| `src/pages/DemandDetailPage.tsx` | 수요 상세 |
| `src/pages/DemandEditPage.tsx` | 수요 수정/삭제 |

## API

- `listDemands(status?)`: `GET /api/demands`
- `getDemand(id)`: `GET /api/demands/{id}`
- `createDemand(data)`: `POST /api/demands`
- `updateDemand(id, data)`: `PATCH /api/demands/{id}`
- `deleteDemand(id)`: `DELETE /api/demands/{id}`

## 데이터 모델

수요는 `species`, `spec`, `quantity`, `needed_at`, `address`, `lat`, `lng`, `budget`, `photo_urls`, `status`를 갖습니다.

`photo_urls`는 선택 필드입니다. 수요 등록 화면은 현재 사진 업로드를 보내지 않고, 시드 데이터 또는 백엔드 데이터에 값이 있으면 목록에서 썸네일로 보여줍니다.

## 화면 흐름

1. 목록은 `listDemands()`로 데이터를 불러옵니다.
2. 구매자만 등록 버튼을 볼 수 있습니다.
3. 등록/수정 화면은 `LocationPicker`로 주소와 좌표를 선택합니다.
4. 상세 화면은 소유자 또는 관리자인 경우 수정/삭제 버튼을 보여줍니다.
5. 수정 후 상세 페이지로 이동하고, 삭제 후 목록으로 이동합니다.

## 리뷰 메모

- 등록/수정 폼에서 숫자는 `Number()`로 변환합니다. 빈 문자열이나 음수에 대한 추가 검증은 약합니다.
- 프론트에서 소유자 버튼 노출을 제어하지만, 최종 권한은 백엔드가 검증합니다.
- 목록/상세/수정 사이의 폼 필드가 중복되어 있습니다. 이후에는 form hook으로 묶을 수 있습니다.
- 수요 상세 화면은 아직 `photo_urls` 갤러리 표시가 약하고 이니셜/그라데이션 중심입니다.

