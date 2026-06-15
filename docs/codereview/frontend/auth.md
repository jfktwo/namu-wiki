# Auth

## 구성 파일

| 파일 | 역할 |
|---|---|
| `src/features/auth/api.ts` | 인증 API 호출 |
| `src/features/auth/useAuth.ts` | 인증 상태 hook |
| `src/pages/LoginPage.tsx` | 로그인 화면 |
| `src/pages/RegisterPage.tsx` | 회원가입 화면 |
| `src/shared/api/client.ts` | 토큰을 요청 헤더에 주입 |

## API 흐름

`auth/api.ts`는 세 가지 요청을 제공합니다.

- `register(data)`: `POST /api/auth/register`
- `login(email, password)`: `POST /api/auth/login`
- `getMe()`: `GET /api/auth/me`

응답은 `access_token`, `token_type`, `user`를 포함합니다.

## useAuth 흐름

`useAuth()`는 앱 인증 상태의 중심입니다.

1. mount 시 `localStorage.token`을 확인합니다.
2. 토큰이 있으면 `/api/auth/me`를 호출합니다.
3. 성공하면 `user` 상태를 채웁니다.
4. 실패하면 토큰을 제거합니다.
5. 로그인/회원가입 성공 시 토큰 저장과 user 갱신을 같이 수행합니다.
6. 로그아웃은 토큰 삭제와 user 초기화를 수행합니다.

## 화면 동작

로그인 화면은 이메일과 비밀번호를 받아 로그인 후 `/`로 이동합니다. 회원가입 화면은 역할, 이메일, 비밀번호, 이름, 전화번호, 지역을 받아 가입 후 `/`로 이동합니다.

회원가입 역할은 `buyer`, `seller`, `admin`을 선택할 수 있습니다. 운영 환경에서는 admin 가입을 일반 사용자에게 열어둘지 별도 정책이 필요합니다.

## 리뷰 메모

- 토큰 저장소가 `localStorage`입니다. 구현은 단순하지만 XSS에 취약할 수 있습니다.
- `useAuth()`가 각 호출 위치마다 독립 state를 만듭니다. 전역 context가 아니므로 여러 컴포넌트에서 동시에 쓰면 상태 싱크가 어긋날 수 있습니다.
- 로그인/회원가입 실패 메시지는 catch-all입니다. 서버 응답 detail을 표시하면 사용자가 원인을 더 잘 알 수 있습니다.
- 현재 화면 텍스트 인코딩이 깨져 있어 UI 카피 복구가 필요합니다.

