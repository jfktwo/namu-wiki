# Admin And Shared UI

## 관리자 화면

### 구성 파일

| 파일 | 역할 |
|---|---|
| `src/features/admin/api.ts` | 사용자 관리 API |
| `src/pages/AdminUsersPage.tsx` | 관리자 사용자 관리 화면 |

### API

- `listUsers()`: `GET /api/admin/users`
- `changeUserRole(userId, role)`: `PATCH /api/admin/users/{user_id}/role`
- `deleteUser(userId)`: `DELETE /api/admin/users/{user_id}`

### 화면 동작

관리자 화면은 사용자 목록을 가져온 뒤 검색어와 역할 필터를 클라이언트에서 적용합니다. 자기 자신은 삭제/역할 변경 UI를 숨깁니다.

## 공통 컴포넌트

| 컴포넌트 | 역할 |
|---|---|
| `Button` | primary, secondary, danger 버튼 스타일과 motion 효과 |
| `Input` | label, required 표시, error 메시지를 포함한 input |
| `Badge` | 상태 배지 |
| `Card` | 카드 컨테이너 |
| `Skeleton` | 로딩 placeholder |
| `ImageUpload` | Supabase Storage 업로드를 통한 이미지 URL 관리 |
| `LocationPicker` | Leaflet 지도 기반 위치 선택 |

## LocationPicker

지도는 `react-leaflet`과 OpenStreetMap tile을 사용합니다. 사용자는 지도를 이동해 중심 좌표를 선택하고, Nominatim reverse geocoding으로 주소를 표시합니다.

현재 위치 버튼은 `navigator.geolocation`을 사용합니다. 권한 거부 시 alert로 안내합니다.

## 리뷰 메모

- 관리자 API는 프론트에서 숨겨도 URL 직접 접근이 가능하므로 백엔드 `require_role("admin")` 검증이 핵심입니다.
- `LocationPicker`는 외부 Nominatim API에 의존합니다. 사용량 정책과 장애 대응을 확인해야 합니다.
- `LocationPicker`의 초기 reverse geocode effect는 lint에서 dependency 누락 경고가 납니다.
- 공통 컴포넌트는 단순하고 재사용성이 좋지만, 폼 검증과 에러 표시 규칙은 아직 화면마다 분산되어 있습니다.

