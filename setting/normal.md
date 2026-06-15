# 기본 설정 (Normal)

## git
echo "# namu" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/jfktwo/namu.git
git push -u origin main


#  see
namu/namucnf1234!

---

## Supabase

| 항목 | 값 |
|------|-----|
| Project URL | https://snfunkfqovrylgnupfto.supabase.co |
| Public Key (sb_publishable) | sb_publishable_OysQF__gTUY-PcD028bNLA_zkJAEIad |
| DATABASE_URL | postgresql://postgres:[YOUR-PASSWORD]@db.snfunkfqovrylgnupfto.supabase.co:5432/postgres |

> `[YOUR-PASSWORD]` = Supabase 프로젝트 생성 시 설정한 DB 비밀번호  
> Secret key (`sb_secret_*`)는 `.env` 파일에만 보관, git 커밋 금지