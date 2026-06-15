# Frontend Overview

## 결론

프론트엔드는 React + Vite + TypeScript 기반의 모바일 우선 SPA입니다. 폴더 구조는 `app`, `pages`, `features`, `shared`로 나뉘며, 페이지 컴포넌트가 화면 상태를 직접 들고 `features/*/api.ts`를 통해 백엔드 REST API를 호출합니다.

역할은 `buyer`, `seller`, `admin` 세 가지입니다. `buyer`는 수요와 매칭/문의, `seller`는 재고와 매칭/문의, `admin`은 사용자 관리와 전체 데이터 조회에 가깝게 동작합니다.

## 주요 파일 지도

| 영역 | 파일 | 역할 |
|---|---|---|
| 앱 시작 | `src/main.tsx`, `src/app/App.tsx` | React 앱 부트스트랩 |
| 라우팅 | `src/app/router.tsx` | URL과 페이지 연결 |
| 레이아웃 | `src/app/Layout.tsx` | 인증 확인, 헤더, 하단 탭, 페이지 전환 |
| API 클라이언트 | `src/shared/api/client.ts` | Axios 인스턴스와 Bearer 토큰 주입 |
| 타입 | `src/shared/api/types.ts` | User, Demand, Stock, Match, Inquiry 타입 |
| 기능 API | `src/features/*/api.ts` | 도메인별 HTTP 호출 |
| 페이지 | `src/pages/*.tsx` | 실제 화면과 로컬 상태 |
| 공통 UI | `src/shared/components/*.tsx` | Button, Input, Badge, 지도, 업로드 등 |

## 데이터 흐름

1. 사용자가 로그인하면 `useAuth()`가 `/api/auth/login`을 호출합니다.
2. 받은 `access_token`은 `localStorage.token`에 저장됩니다.
3. `shared/api/client.ts`의 Axios interceptor가 모든 요청에 `Authorization: Bearer <token>`을 붙입니다.
4. 각 페이지는 `features` API 함수를 호출해서 데이터를 가져오고, `useState`로 화면 상태를 관리합니다.
5. 생성/수정/삭제 후에는 목록 페이지로 이동하거나 데이터를 다시 로드합니다.

## 기능 단위

- 인증: 로그인, 회원가입, 현재 사용자 조회
- 수요: 구매자가 나무 수요를 등록하고 관리
- 재고: 공급자가 재고와 이미지를 등록하고 관리
- 매칭: 수요와 재고 조건이 맞는 후보 조회
- 문의: 매칭 후보에 대해 문의 요청, 수락, 거절, 취소
- 관리자: 사용자 목록 조회, 역할 변경, 사용자 삭제

## 코드리뷰 주요 발견

1. **높음: 화면 문자열 인코딩이 깨져 있습니다.**
   여러 TSX 파일의 한글 텍스트가 `?섎Т`, `臾몄쓽`처럼 깨져 있습니다. 기능 로직과 별개로 사용자 화면 품질에 직접 영향을 줍니다. 파일 저장 인코딩 또는 이전 생성 과정에서 손상된 것으로 보이며, UI 카피를 다시 정리해야 합니다.

2. **중간: 인증 리다이렉트가 render 중에 실행됩니다.**
   `Layout.tsx`에서 `!user`일 때 `navigate('/login')`을 render 흐름 안에서 호출합니다. React 관점에서는 effect나 `<Navigate />`가 더 안정적입니다.

3. **중간: 일부 상태 변경이 effect 본문에서 동기적으로 실행되어 lint 실패가 납니다.**
   `useAuth`, `AdminUsersPage`, `InquiryListPage`, `MatchListPage` 등에서 React Hooks lint가 실패합니다. 빌드는 되지만 품질 게이트로 쓰려면 패턴 정리가 필요합니다.

4. **중간: 권한 제어가 프론트 표시와 백엔드 검증에 나뉘어 있습니다.**
   프론트는 역할별 탭과 버튼 노출을 제어하지만, 실제 권한은 백엔드가 최종 검증합니다. 이 구조는 맞지만 프론트에서 접근 불가 페이지를 명시적으로 막는 라우트 가드는 아직 약합니다.

5. **낮음: 페이지 컴포넌트가 API 호출, 폼 상태, 표시 로직을 모두 갖고 있습니다.**
   MVP 단계에서는 이해하기 쉽지만, 화면이 커지면 hook 또는 form helper로 분리하는 편이 유지보수에 좋습니다.

## 검증 상태

- `npm.cmd test`: 통과
- `npm.cmd run build`: 통과
- `npm.cmd run lint`: 기존 React Hooks lint 이슈로 실패

