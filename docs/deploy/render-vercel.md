# Render + Vercel 임시 서비스 배포 가이드

나무 장터 MVP를 외부에서 접근 가능한 환경에 올리는 실전 절차다.

```
DB: Supabase (이미 운영 중)
백엔드: Render (FastAPI)
프론트: Vercel (React/Vite)
```

---

## 사전 준비

### GitHub에 최신 코드 올리기

아직 push 안 한 경우 먼저 실행한다.

```bash
git add .
git commit -m "deploy: render + vercel 배포 준비"
git push origin main
```

### 필요한 환경변수 미리 확인

Supabase 대시보드 → Project Settings → API 에서 값을 복사해둔다.

| 항목 | 위치 |
|------|------|
| `SUPABASE_URL` | Project URL |
| `SUPABASE_ANON_KEY` | anon / public |
| `SUPABASE_SECRET_KEY` | service_role (절대 프론트에 넣으면 안 됨) |
| `DATABASE_URL` | Settings → Database → Connection string (URI) |

---

## 1단계 — 백엔드: Render

### 가입 및 연결

1. [render.com](https://render.com) 접속 → GitHub 계정으로 가입
2. 대시보드에서 **New → Web Service** 클릭
3. GitHub 저장소 선택 → **Connect**

### 서비스 기본 설정

| 항목 | 값 |
|------|-----|
| Name | `namu-backend` |
| Region | Singapore (Asia 기준 가장 빠름) |
| Branch | `main` |
| Root Directory | `backend` |
| Runtime | `Python 3` |
| Build Command | `pip install -r requirements.txt` |
| Start Command | `uvicorn app.main:app --host 0.0.0.0 --port $PORT` |
| Instance Type | **Free** (슬립 있음) |

> render.yaml 이 프로젝트 루트에 있으면 위 설정 대부분이 자동으로 채워진다.

### 환경변수 등록

**Environment** 탭 → **Add Environment Variable** 에서 아래 값을 입력한다.

| Key | Value |
|-----|-------|
| `SUPABASE_URL` | `https://xxxxx.supabase.co` |
| `SUPABASE_ANON_KEY` | `eyJxxx...` |
| `SUPABASE_SECRET_KEY` | `eyJxxx...` (service_role) |
| `DATABASE_URL` | `postgresql://postgres:...` |
| `SECRET_KEY` | 랜덤 32자 이상 문자열 |
| `ALLOWED_ORIGINS` | `*` (임시, Vercel URL 확인 후 교체) |

> `SECRET_KEY` 생성: 터미널에서 `openssl rand -hex 32` 실행

### 배포 확인

- **Create Web Service** 클릭 후 로그 확인
- 배포 완료 후 상단에 URL 표시됨 (예: `https://namu-backend.onrender.com`)
- `https://namu-backend.onrender.com/health` 접속해서 `{"status":"ok"}` 응답 확인

---

## 2단계 — 프론트엔드: Vercel

### 가입 및 연결

1. [vercel.com](https://vercel.com) 접속 → GitHub 계정으로 가입
2. **Add New Project** → GitHub 저장소 선택 → **Import**

### 프로젝트 설정

| 항목 | 값 |
|------|-----|
| Framework Preset | `Vite` |
| **Root Directory** | `frontend` ← 반드시 변경 |
| Build Command | `npm run build` (자동 감지) |
| Output Directory | `dist` (자동 감지) |

### 환경변수 등록

**Environment Variables** 섹션에서 추가한다.

| Key | Value |
|-----|-------|
| `VITE_API_BASE_URL` | `https://namu-backend.onrender.com` (Render URL) |

### 배포 확인

- **Deploy** 클릭
- 빌드 완료 후 URL 표시됨 (예: `https://namu-market.vercel.app`)
- 해당 URL로 접속해서 로그인 화면 확인

---

## 3단계 — CORS 업데이트

Vercel 배포 후 나온 도메인을 Render 환경변수에 반영해야 한다.

Render 대시보드 → 해당 서비스 → **Environment** → `ALLOWED_ORIGINS` 값 수정:

```
ALLOWED_ORIGINS=https://namu-market.vercel.app
```

저장하면 Render가 자동으로 재배포한다.

여러 도메인이 필요하면 쉼표로 구분한다.

```
ALLOWED_ORIGINS=https://namu-market.vercel.app,https://namu-market-git-main.vercel.app
```

---

## 4단계 — 동작 확인

| 확인 항목 | 방법 |
|----------|------|
| 백엔드 헬스 | `https://namu-backend.onrender.com/health` |
| 프론트 접속 | Vercel URL 브라우저 열기 |
| 로그인 | user1@test.com / (설정된 비밀번호) |
| 역할 분기 | seller 로그인 시 재고 목록 첫 화면 확인 |
| 매칭/문의 흐름 | 매칭 → 문의 등록 → 수락 확인 |

---

## 주의 사항

### Render 무료 플랜 슬립

무료 플랜은 **15분 비활성 시 자동 슬립**된다. 슬립 후 첫 요청은 30초 정도 걸릴 수 있다.

지속 운영이 필요하면 유료 플랜($7/월)으로 전환한다.

### 재배포 방법

코드 수정 후 `git push origin main` 하면 Render와 Vercel 모두 자동으로 재배포된다.

### 환경변수 변경

Render나 Vercel에서 환경변수를 바꾸면 수동으로 **Redeploy** 버튼을 눌러야 반영된다.

---

## 배포 후 도메인 구조 요약

```
Supabase
  https://xxxxx.supabase.co
  └─ PostgreSQL DB + Storage

Render (백엔드)
  https://namu-backend.onrender.com
  └─ FastAPI
  └─ /health /api/...

Vercel (프론트엔드)
  https://namu-market.vercel.app
  └─ React/Vite 빌드
  └─ VITE_API_BASE_URL → Render URL
```
