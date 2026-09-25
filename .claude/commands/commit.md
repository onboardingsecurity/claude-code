---
description: 현재 변경사항을 git add, git commit 하고 한 줄짜리 prefix 커밋 메시지를 자동 생성합니다.
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*), Bash(git diff:*), Bash(git branch:*), 
---

인자 없이 실행되는 커맨드입니다. `$ARGUMENTS`는 무시하세요.

## 목표
현재 브랜치의 변경사항을 `git add`, `git commit`으로 커밋한다. 커밋 메시지는
`feat:`, `fix:`, `docs:`, `debug:`, `db:`, `api:`, `test:` 등 적절한 prefix +
작업 내용을 요약한 한 줄 메시지로 작성한다.

## 토큰 절약 지침 (중요)
- 전체 diff 본문(`git diff` 전체 출력, 파일 내용 전체)을 읽거나 출력하지 마라.
- 변경 내역 파악에는 아래처럼 **요약 정보만** 사용한다:
  - `git status --short` — 변경/추가/삭제된 파일 목록
  - `git diff --stat --cached` (add 이후) — 파일별 변경 라인 수 요약
  - 필요 시 `git diff --cached -- <특정 파일>`처럼 **범위를 좁혀서** 최소한만 확인 (예: prefix 판단이 애매한 파일 1~2개 정도)
- 커밋 완료 후 결과 확인은 `git log -1 --oneline` 정도로 짧게 한다. `git show`, 전체 로그 등 불필요한 출력은 하지 않는다.
- 사용자에게는 최종 커밋 해시와 메시지만 한 줄로 보고한다. 중간 명령 실행 결과를 길게 나열하지 마라.

## 절차
1. `git status --short` 로 변경 파일 목록만 빠르게 확인한다.
2. `git add -A` 로 전체 변경사항을 스테이징한다.
3. `git diff --cached --stat` 로 스테이징된 변경의 요약(파일명, +/- 라인 수)만 확인한다.
   - 파일 경로/확장자만으로 prefix 판단이 가능하면 이 단계만으로 충분하다.
   - 애매한 경우에만 해당 파일에 한해 좁은 범위로 `git diff --cached -- <path>` 를 추가로 확인한다.
4. 변경 성격에 맞는 prefix를 정한다 (아래 규칙 참고).
5. 작업 내용을 요약한 한글 또는 영어 한 줄 커밋 메시지를 작성한다. 형식: `<prefix> <한 줄 요약>`
   - 예: `feat: 로그인 페이지 소셜 로그인 버튼 추가`
   - 예: `fix: 커밋 메시지 prefix 판별 로직 오류 수정`
   - 본문(body)은 작성하지 않는다. 제목 한 줄만.
6. `git commit -m "<prefix> <요약>"` 실행.
7. `git log -1 --oneline` 으로 결과만 짧게 확인 후 사용자에게 보고한다.

## Prefix 판단 규칙 (우선순위 순)
- `test:` — 테스트 파일(`*test*`, `*spec*`, `__tests__/`)만 변경된 경우
- `docs:` — `README`, `*.md`, `docs/` 등 문서만 변경된 경우
- `db:` — 마이그레이션, 스키마, DB 모델/쿼리 관련 변경
- `api:` — API 라우트/엔드포인트/컨트롤러 관련 변경
- `fix:` — 버그 수정으로 보이는 변경 (에러 처리, 조건문 수정, 오탈자성 로직 수정 등)
- `debug:` — 로깅/디버깅 코드 추가·수정 위주의 변경
- `feat:` — 새로운 기능/파일 추가
- 위에 명확히 해당하지 않으면 변경 규모와 맥락상 가장 적절한 prefix를 판단해서 사용한다 (`refactor:`, `chore:` 등도 필요하면 사용 가능).

## 주의사항
- 스테이징/커밋할 변경사항이 없으면(`git status --short` 결과가 비어있으면) 커밋을 시도하지 말고 "커밋할 변경사항이 없습니다"라고만 보고한다.
- 커밋 전에 `.env`, credential 파일처럼 민감해 보이는 파일이 포함되어 있는지 `git status --short` 결과에서 파일명만으로 확인하고, 의심되면 사용자에게 알린다.
- 커밋 메시지, 브랜치 삭제, force push 등 위험한 동작은 하지 않는다. 이 커맨드는 add + commit만 수행한다.
