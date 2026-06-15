# 패키징/배포 가이드

## 결론

나무 장터는 다음 구조가 가장 적합하다.

```text
프론트 웹: Vercel 같은 정적 웹 배포 서비스
모바일 앱: Capacitor로 React/Vite 프론트 번들 패키징
백엔드: FastAPI를 별도 웹서버에 배포
DB/Storage: Supabase 유지
```

한 줄로 정리하면:

```text
웹도 되고 앱도 되는 하이브리드 구조.
단, 백엔드는 앱 안에 넣지 않고 반드시 서버에 둔다.
```

## 추천 아키텍처

```text
[웹 브라우저]
Vercel에 배포된 React/Vite 프론트
        |
        | HTTPS API 호출
        v
[백엔드 서버]
FastAPI
        |
        v
[Supabase]
PostgreSQL DB + Storage
```

```text
[모바일 앱]
Capacitor 껍데기 + React/Vite 프론트 번들
        |
        | HTTPS API 호출
        v
[백엔드 서버]
FastAPI
        |
        v
[Supabase]
PostgreSQL DB + Storage
```

프론트 코드는 하나만 유지한다. 같은 `frontend` 코드를 웹에서는 Vercel에 올리고, 앱에서는 Capacitor로 빌드해서 넣는다.

## 왜 백엔드는 앱 안에 넣으면 안 되나?

백엔드는 앱에 넣으면 안 된다.

이유:

- Supabase secret key가 노출될 수 있다.
- JWT secret, 서버 권한 로직, 관리자 API가 노출될 수 있다.
- 앱마다 백엔드가 따로 있으면 데이터 동기화가 불가능하다.
- 사용자 권한 검증과 DB 처리는 중앙 서버가 담당해야 한다.

구분:

| 위치 | 넣는 것 |
|---|---|
| 앱 안 | 화면, UI, 프론트 코드, Capacitor 설정 |
| 서버 | FastAPI, 비밀키, 권한 검증, DB 처리, 이미지 업로드 API |
| Supabase | DB, Storage, public 이미지 URL |

## 선택지 비교

### 1. 백엔드만 서버, 프론트는 앱에 번들

```text
앱 안에 React 빌드 파일 포함
백엔드는 Render/Railway/Fly.io/EC2 등에 배포
```

장점:

- 앱이 진짜 앱처럼 동작한다.
- 첫 화면이 빠르다.
- 앱스토어/플레이스토어 심사에 자연스럽다.
- 위치, 카메라, 푸시 같은 네이티브 기능 확장에 좋다.

단점:

- 프론트 수정 후 앱 재배포가 필요할 수 있다.
- 웹 버전과 앱 버전 관리가 필요하다.

### 2. 모든 화면을 웹서버에 올리고 앱은 WebView 껍데기만

```text
앱 = https://도메인 을 여는 WebView
프론트 = Vercel
백엔드 = API 서버
```

장점:

- 웹만 수정하면 앱에도 바로 반영된다.
- 초기 개발은 가장 단순하다.

단점:

- 인터넷이 느리거나 끊기면 앱이 빈 화면이 될 수 있다.
- 앱스토어 심사에서 단순 웹 래핑으로 보일 수 있다.
- 네이티브 기능 연동이 애매해질 수 있다.
- 앱 품질이 낮아 보일 수 있다.

### 3. 웹 배포 + 앱 번들 + 나중에 Live Update

```text
웹: Vercel
앱: Capacitor 번들
백엔드: FastAPI 서버
추후: Live Update/Appflow류로 JS/CSS 업데이트
```

추천안이다.

장점:

- 웹과 앱을 둘 다 제공할 수 있다.
- 앱은 기본 화면 번들을 포함해서 안정적이다.
- 나중에 작은 프론트 수정은 Live Update 방식으로 빠르게 반영할 수 있다.

## Vercel이란?

Vercel은 프론트엔드 웹페이지를 인터넷에 올려주는 서비스다.

로컬 개발 주소:

```text
http://localhost:5173
```

Vercel 배포 후 예시:

```text
https://namu-market.vercel.app
```

Vercel은 앱스토어 배포가 아니다. 웹 브라우저에서 접속할 수 있는 프론트 웹 배포다.

## 무료 시험판 기준

MVP/시험판은 대부분 무료 또는 저비용으로 가능하다.

| 항목 | 시험판 비용 |
|---|---|
| 프론트 웹 배포 | Vercel Free 가능 |
| 백엔드 배포 | Render/Railway/Fly.io 무료 또는 저가 플랜 |
| DB/Storage | Supabase Free 가능 |
| 앱 패키징 | Capacitor 무료 |
| Android APK 직접 설치 테스트 | 무료 |
| Google Play 등록 | 1회 25달러 |
| Apple Developer 등록 | 연 99달러 |

가장 먼저 비용이 들 가능성이 있는 항목:

1. Google Play 등록비
2. 백엔드 서버를 24시간 안정적으로 켜두는 비용
3. Supabase Storage/Egress 증가
4. Apple App Store 등록비

## 배포 순서

### 1단계: 백엔드 배포

FastAPI 백엔드를 먼저 인터넷에서 접근 가능한 서버에 올린다.

후보:

- Render
- Railway
- Fly.io
- AWS Lightsail/EC2

배포 후 API 주소 예시:

```text
https://api.namu-market.com
```

필요 환경변수:

```env
SUPABASE_URL=...
SUPABASE_SECRET_KEY=...
SECRET_KEY=...
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=...
```

주의:

- `SUPABASE_SECRET_KEY`는 절대 프론트나 앱에 넣으면 안 된다.
- CORS에 프론트 웹 주소와 앱에서 사용할 origin을 맞춰야 한다.

### 2단계: 프론트 웹 배포

Vercel에서 GitHub repo를 import한다.

우리 프로젝트 기준 설정:

| 설정 | 값 |
|---|---|
| Framework Preset | Vite |
| Root Directory | `frontend` |
| Build Command | `npm run build` |
| Output Directory | `dist` |

환경변수:

```env
VITE_API_BASE_URL=https://배포된-백엔드주소
```

중요:

```text
VITE_API_BASE_URL=http://localhost:8000
```

이 값은 배포 환경에서 쓰면 안 된다. 배포된 백엔드 HTTPS 주소를 넣어야 한다.

### 3단계: Capacitor 앱 패키징

프론트 폴더에서 진행한다.

```powershell
cd frontend
npm.cmd install @capacitor/core @capacitor/cli @capacitor/android
npx.cmd cap init "나무 장터" "com.namu.market" --web-dir=dist
npm.cmd run build
npx.cmd cap add android
npx.cmd cap sync android
npx.cmd cap open android
```

이미 Capacitor가 초기화되어 있으면 `cap init`과 `cap add android`는 반복하지 않는다.

일반 수정 후에는 보통 이 흐름이다.

```powershell
cd frontend
npm.cmd run build
npx.cmd cap sync android
npx.cmd cap open android
```

### 4단계: Android 테스트

Android Studio에서 실행하거나 APK를 빌드해서 직접 설치한다.

시험판 배포는 다음 방식이 가능하다.

- 개발자 휴대폰에 직접 APK 설치
- 내부 테스트 APK 공유
- Google Play Console 내부 테스트

Google Play에 올리려면 개발자 계정 등록비가 필요하다.

### 5단계: iOS 테스트

iOS는 Mac과 Xcode가 필요하다.

```powershell
cd frontend
npm.cmd install @capacitor/ios
npm.cmd run build
npx.cmd cap add ios
npx.cmd cap sync ios
npx.cmd cap open ios
```

Apple App Store/TestFlight 배포는 Apple Developer Program이 필요하다.

## 환경변수 전략

개발:

```env
VITE_API_BASE_URL=http://localhost:8000
```

Android Emulator:

```env
VITE_API_BASE_URL=http://10.0.2.2:8000
```

실제 배포/앱:

```env
VITE_API_BASE_URL=https://api.namu-market.com
```

실제 휴대폰에서 로컬 백엔드를 보려면 PC의 LAN IP를 써야 한다.

예:

```env
VITE_API_BASE_URL=http://192.168.0.25:8000
```

운영 배포에서는 HTTPS를 사용하는 것이 좋다.

## 현재 프로젝트에서 필요한 체크리스트

- [ ] 백엔드 배포 서비스 선택
- [ ] 백엔드 환경변수 등록
- [ ] 백엔드 CORS origin 수정
- [ ] 프론트 Vercel 배포
- [ ] Vercel에 `VITE_API_BASE_URL` 등록
- [ ] Capacitor 초기화
- [ ] Android 앱 빌드
- [ ] APK 직접 설치 테스트
- [ ] 앱에서 로그인/목록/이미지/문의 동작 확인
- [ ] Supabase Usage 확인

## 추천 최종 구조

```text
Supabase
  - DB
  - Storage

FastAPI Backend
  - Render/Railway/Fly.io/EC2
  - 비밀키와 권한 검증 담당

React Frontend Web
  - Vercel
  - 브라우저 접속용

Mobile App
  - Capacitor
  - React 빌드 결과 포함
  - 백엔드 API 호출
```

이 구조가 시험판부터 운영 전환까지 가장 자연스럽다.

