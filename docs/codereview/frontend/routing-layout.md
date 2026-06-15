# Routing And Layout

## 라우팅 구조

라우팅은 `src/app/router.tsx`에서 `createBrowserRouter`로 정의합니다.

| 경로 | 페이지 | 설명 |
|---|---|---|
| `/login` | `LoginPage` | 로그인 |
| `/register` | `RegisterPage` | 회원가입 |
| `/` | `Layout` | 인증 사용자용 공통 레이아웃 |
| `/demands` | `DemandListPage` | 수요 목록 |
| `/demands/new` | `DemandCreatePage` | 수요 등록 |
| `/demands/:id` | `DemandDetailPage` | 수요 상세 |
| `/demands/:id/edit` | `DemandEditPage` | 수요 수정 |
| `/stocks` | `StockListPage` | 재고 목록 |
| `/stocks/new` | `StockCreatePage` | 재고 등록 |
| `/stocks/:id` | `StockDetailPage` | 재고 상세 |
| `/stocks/:id/edit` | `StockEditPage` | 재고 수정 |
| `/matches` | `MatchListPage` | 매칭 후보 |
| `/matches/:id` | `MatchDetailPage` | 매칭 상세 |
| `/inquiries` | `InquiryListPage` | 문의 목록 |
| `/admin/users` | `AdminUsersPage` | 사용자 관리 |

## Layout 역할

`src/app/Layout.tsx`는 로그인 이후 모든 화면의 공통 껍데기입니다.

- `useAuth()`로 현재 사용자와 로딩 상태를 가져옵니다.
- 사용자가 없으면 로그인으로 이동합니다.
- 상단 헤더에 앱명, 사용자명, 역할, 로그아웃 버튼을 표시합니다.
- 하단 탭 메뉴를 역할별로 다르게 구성합니다.
- `Outlet`으로 하위 페이지를 렌더링합니다.
- Framer Motion으로 페이지 전환 애니메이션을 적용합니다.

## 역할별 하단 탭

| 역할 | 탭 |
|---|---|
| buyer | 수요, 매칭, 문의 |
| seller | 재고, 매칭, 문의 |
| admin | 수요, 재고, 매칭, 사용자 관리 |

## 리뷰 메모

- `navigate('/login')`이 render 중 호출됩니다. `<Navigate to="/login" replace />` 또는 `useEffect`로 바꾸는 것이 React 패턴에 더 맞습니다.
- `useRef`로 이전 path depth를 저장하고 render 중 값을 읽고 씁니다. 현재 lint가 이 부분을 지적합니다.
- 역할별 메뉴는 명확하지만, URL 직접 접근에 대한 프론트 라우트 가드는 약합니다. 백엔드 권한 검증이 최종 방어선입니다.

