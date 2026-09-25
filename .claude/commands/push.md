---
description: 현재 브랜치를 원격 저장소로 git push 합니다.
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*), Bash(git diff:*), Bash(git branch:*), Bash(git push:*),
---

인자 없이 실행되는 커맨드입니다. `$ARGUMENTS`는 무시하세요.

## 목표
현재 브랜치의 커밋을 원격(origin)으로 `git push` 한다. 이 커맨드는 push만 수행하며,
add/commit은 하지 않는다 (필요하면 `/commit`을 먼저 실행한다).

## 토큰 절약 지침 (중요)
- `git log -p`, `git diff` 등 커밋 내용 전체를 읽거나 출력하지 마라.
- 아래처럼 **요약 정보만** 사용한다:
  - `git status -sb` — 현재 브랜치, 추적 브랜치(upstream), ahead/behind 여부 확인
  - `git log --oneline @{u}..HEAD` (upstream이 있는 경우) — push될 커밋 목록만 짧게 확인
- push 실행 결과(성공 로그)는 요약해서 한두 줄로만 보고한다. 전체 출력을 그대로 나열하지 않는다.

## 절차
1. `git status -sb` 로 현재 브랜치와 upstream 설정 여부, ahead/behind 상태를 확인한다.
2. 커밋되지 않은 변경사항(untracked/modified)이 있어도 이 커맨드는 무시한다 — push 대상은 이미 커밋된 내역뿐이다.
3. push할 커밋이 없으면(ahead 0) "push할 커밋이 없습니다"라고 보고하고 종료한다.
4. upstream이 이미 설정되어 있으면 `git push` 실행.
   upstream이 없으면 `git push -u origin <현재 브랜치명>` 실행.
5. 현재 브랜치가 `main` 또는 `master`인 경우, push 실행 전 사용자에게 한 줄로 확인을 받는다.
   그 외 브랜치는 바로 진행한다.
6. push 결과를 커밋 개수와 브랜치명 정도로 짧게 요약해서 보고한다.

## 주의사항
- `--force`, `--force-with-lease`, `-f` 등 강제 push는 사용자가 명시적으로 요청하지 않는 한 절대 사용하지 않는다.
- 원격 브랜치를 삭제하거나(`git push origin --delete`) 다른 브랜치로 push하는 동작은 하지 않는다. 항상 현재 브랜치 → 동일 이름의 원격 브랜치로만 push한다.
- push 도중 reject(비-fast-forward 등) 에러가 나면 임의로 `--force`나 `git pull --rebase` 등을 실행하지 말고, 에러 내용을 사용자에게 보고하고 어떻게 처리할지 확인한다.
