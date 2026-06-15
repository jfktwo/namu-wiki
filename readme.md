# 나무 장터 프로젝트 Wiki

---

## userguide

- [사용자 가이드](userguide/readme.md) - 실제 화면 캡처와 함께 정리한 관리자, 수요자, 공급자 역할별 사용 방법

## feature

- [기능 발전 체크리스트](feature/readme.md) - MVP 안정화, 스키마 변경 후보, 매칭 고도화, 수익화, 웹/앱 발전 단계를 체크리스트로 정리

## design

- [디자인 가이드](design/design-guide.md) - 초록색 점 패턴, 카드/버튼/사진 중심 UI, 모바일 업무 화면 기준 정리
- [디자인 감사 자료](design/audit/README.md) - 로그인 후 실제 화면 캡쳐, 페이지별 진단, 구성요소별 개선안, 개편 실행 계획
### business
- [과금 아이디어](business/monetization-ideas.md) - 공급자 구독, 리드 과금, 거래 수수료, 시세 리포트, SaaS 확장 아이디어

### deploy
- [패키징/배포 가이드](deploy/packaging-guide.md) - Vercel, 백엔드 서버, Supabase, Capacitor 앱 패키징 구조
- [Render + Vercel 실전 배포](deploy/render-vercel.md) - GitHub → Render(백엔드) → Vercel(프론트) 단계별 절차, 환경변수, CORS 설정

수목 수요자(조경업체)와 공급자(묘목장)를 연결하는 하이브리드 앱 개발 문서입니다.

---

## 목차

### start — 프로젝트 기초
- [01. 구조잡기](start/01.구조잡기.md) — 서비스 개념, 기능 정의, 기술 스택, 단계별 계획
- [02. 소프트웨어 구조](start/02.소프트웨어 구조.md) — 프론트/백엔드 분리 구조, 기술 스택, 폴더 구조, API 초안
- [03. 개발환경구성](start/03.개발환경구성.md) — 로컬 개발환경, 프론트엔드/백엔드 초기화, 환경 변수
- [04. 요구사항 분석](start/04.요구사항%20분석.md) — MVP 기능 범위, 권한, 매칭 조건, 검증 기준
- [05. 디자인 시안](start/05.디자인%20시안.md) — 초록색 점 패턴 기반 화면 디자인 방향과 주요 화면 시안

### workflow — 작업 흐름
- [워크플로우](workflow.md) — 역할별 에이전트, 진행 흐름, 완료 기준

### setting — 설정
- [normal](setting/normal.md) — 기본 설정
- [DB 마이그레이션 실행](setting/db-migration.md) — SQL Editor / Python 스크립트 실행법, 실패 대처법

### test — 테스트
- [목차 & 추가 필요 항목](test/readme.md) — 테스트 현황 및 작성 예정 목록
- [01. 기본사항](test/01.기본사항.md) — 테스트 구조, 실행 방법, 환경 준비
- [02. 테스트 내용](test/02.테스트내용.md) — 현재 작성된 테스트 케이스 목록

### future — 추후 작업
- [목차](future/readme.md) — 논의·결정됐지만 미구현된 기능 목록
- [01. 동영상 업로드](future/01.video.md) — Cloudflare R2 기반 동영상 업로드 계획
