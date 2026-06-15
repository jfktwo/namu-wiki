# 디자인 감사 자료

로그인 후 실제 화면을 캡쳐하고, 현재 디자인 상태를 페이지별/구성요소별로 정리한 자료다.

## 산출물

- [캡쳐 인벤토리](capture-inventory.md)  
  로그인, 회원가입, 수요자, 공급자, 관리자 화면의 모바일/데스크톱 캡쳐 목록

- [현재 디자인 진단](current-design-audit.md)  
  화면별 관찰 내용과 개선 방향

- [구성요소별 디자인 개선안](component-improvement-plan.md)  
  레이아웃, 카드, 버튼, 사진, 배지, 문구 기준의 개선안

- [디자인 개편 실행 계획](redesign-execution-plan.md)  
  실제 프론트엔드 개편을 어떤 순서로 진행할지 정리한 계획

## 캡쳐 범위

- 공개 화면: 로그인, 회원가입
- 수요자 화면: 수요 목록, 수요 등록, 매칭 목록, 문의 목록
- 공급자 화면: 재고 목록, 재고 등록, 매칭 목록, 문의 목록
- 관리자 화면: 대시보드, 사용자, 수요, 재고, 매칭, 문의

각 화면은 모바일과 데스크톱 뷰포트로 캡쳐했다.

## 스크린샷 폴더

- [screenshots](screenshots)

## 다시 생성하는 방법

프론트엔드와 백엔드가 실행 중인 상태에서 아래 명령을 실행한다.

```powershell
node tools\design_audit.mjs
```

기본 주소:

- 프론트엔드: `http://localhost:5173`
- 백엔드: `http://localhost:8000`

다른 주소를 쓰려면 환경 변수를 지정한다.

```powershell
$env:DESIGN_AUDIT_BASE_URL="http://localhost:5173"
$env:DESIGN_AUDIT_API_URL="http://localhost:8000"
node tools\design_audit.mjs
```

## 다음 작업

1. 깨진 한글 문구 복구
2. 공통 컴포넌트 기준 정리
3. 재고/수요/매칭 카드 정보 구조 통일
4. 사진 중심 카드 개편
5. 상태 배지 체계 정리
6. 같은 스크립트로 전/후 화면 재캡쳐
