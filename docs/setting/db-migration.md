# DB 마이그레이션 실행 방법

Supabase에 SQL 마이그레이션을 적용하는 방법입니다.

---

## 방법 1 — Supabase SQL Editor (권장, 간단)

1. [supabase.com](https://supabase.com) → 프로젝트 → 좌측 메뉴 **SQL Editor**
2. `database/migrations/` 에서 실행할 `.sql` 파일 내용을 복사
3. 붙여넣기 후 **Run** 클릭

**주의**: 대용량 마이그레이션(PostGIS extension 활성화, GiST 인덱스 생성 등)은 10분 이상 걸릴 수 있습니다. 네트워크가 끊겨도 Supabase 서버에서 계속 실행되므로 중간에 멈추지 마세요.

---

## 방법 2 — Python 스크립트 (psql 없을 때)

로컬에 `psql`이 없는 경우, 백엔드 가상환경의 Python으로 실행합니다.

### 준비

```powershell
cd c:\...\proj1\backend
.\.venv\Scripts\pip.exe install psycopg2-binary
```

### 실행 스크립트 작성 (`run_migration.py`)

```python
import psycopg2, pathlib, sys

DATABASE_URL = "postgresql://postgres:<비밀번호>@db.<project>.supabase.co:5432/postgres"
SQL_PATH = pathlib.Path(__file__).parent / "database" / "migrations" / "011_xxx.sql"

sql = SQL_PATH.read_text(encoding="utf-8")

conn = psycopg2.connect(DATABASE_URL)
conn.autocommit = True
cur = conn.cursor()
try:
    cur.execute(sql)
    print("SUCCESS")
except Exception as e:
    print(f"ERROR: {e}", file=sys.stderr)
    sys.exit(1)
finally:
    cur.close()
    conn.close()
```

```powershell
.\.venv\Scripts\python.exe ..\run_migration.py
```

실행 후 `run_migration.py` 파일은 삭제하세요 (DB 비밀번호 포함).

### DATABASE_URL 위치

`.env` 파일에 있습니다:

```
DATABASE_URL=postgresql://postgres:<비밀번호>@db.<project-ref>.supabase.co:5432/postgres
```

---

## 마이그레이션 실패 시 대처

### 함수 반환타입 변경 오류

```
ERROR: cannot change return type of existing function
HINT: Use DROP FUNCTION find_matches_for_buyer(...) first.
```

`CREATE OR REPLACE FUNCTION`은 반환 컬럼이 바뀌면 실패합니다. 해결:

```sql
DROP FUNCTION IF EXISTS find_matches_for_buyer(uuid, float);
DROP FUNCTION IF EXISTS find_matches_for_seller(uuid, float);
DROP FUNCTION IF EXISTS find_all_matches(float);
```

이후 함수를 다시 `CREATE FUNCTION`으로 생성합니다.

### 네트워크 단절로 중간 실패

DDL(컬럼 추가, 인덱스 생성)은 일부만 완료된 상태가 될 수 있습니다.  
재실행 SQL에는 반드시 `IF NOT EXISTS` / `IF EXISTS`를 사용해 멱등성을 확보하세요:

```sql
ALTER TABLE stocks ADD COLUMN IF NOT EXISTS location geography(Point, 4326);
CREATE INDEX IF NOT EXISTS idx_stocks_location ON stocks USING GIST(location);
DROP TRIGGER IF EXISTS stocks_sync_location ON stocks;
```

데이터 재처리는 WHERE로 범위를 제한합니다:

```sql
-- 이미 채워진 row는 건드리지 않음
UPDATE stocks SET location = ... WHERE location IS NULL;
```

### 실제 사례: 011 통합 마이그레이션

009(PostGIS, 네트워크 단절로 미완) + 010(함수 반환타입 오류) 문제가 겹쳐 011로 통합 처리했습니다.  
→ `database/migrations/011_postgis_and_scoring_fix.sql` 참고

---

## 마이그레이션 파일 목록

`todo/DB마이그레이션.md` 에서 실행 상태를 관리합니다.
