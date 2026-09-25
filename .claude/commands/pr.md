---
description: commit, push 커맨드를 순서대로 실행한 뒤 GitHub PR을 생성합니다.
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*), Bash(git diff:*), Bash(git branch:*), Bash(git push:*), Bash(git log:*), Bash(gh pr:*), Bash(gh repo:*)
---

인자로 base 브랜치(PR이 병합될 대상 브랜치)를 받는다: `$ARGUMENTS`
- 인자가 없으면 base 브랜치는 저장소의 기본 브랜치(main 등)이다.
- 인자가 있으면 그 값을 base 브랜치로 사용한다.
- head 브랜치(PR에 담길 변경사항이 있는 브랜치)는 항상 **현재 브랜치**이다. 체크아웃을 바꾸지 않는다.

## 목표
현재 브랜치의 변경사항을 커밋/푸시한 뒤, `head=현재 브랜치` → `base=$ARGUMENTS 또는 기본 브랜치`
로 향하는 GitHub PR을 생성한다.

## 핵심 원칙 — commit/push 커맨드 재사용
변경사항 스테이징, 커밋 메시지 작성, push 로직은 이미 `commit`, `push` 스킬에 정의되어 있다.
**이 로직을 다시 만들지 말고 Skill 도구로 그대로 호출**해서 토큰을 절약한다.

1. `Skill(skill: "commit")` 호출 — 변경사항을 add + commit (커밋할 게 없으면 스킵됨).
2. `Skill(skill: "push")` 호출 — 현재 브랜치를 origin에 push (push할 게 없으면 스킵됨).
3. 위 두 스킬 실행 중 출력된 결과(커밋 메시지, push 여부)를 그대로 재사용한다.
   두 스킬이 이미 확인한 `git status`, `git diff` 등을 이 커맨드에서 다시 조회하지 않는다.

## 토큰 절약 지침
- PR 본문 작성을 위해 diff 전체를 읽지 않는다. base와의 커밋 목록만 짧게 확인한다:
  - `git log --oneline <base>..HEAD`
- PR 생성/조회 결과는 URL 한 줄 정도로만 보고한다. `gh pr view` 풀 출력 전체를 붙여넣지 않는다.

## 절차
1. base 브랜치를 정한다.
   - `$ARGUMENTS`가 있으면 그 값을 사용한다.
   - 없으면 `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` 로 기본 브랜치를 확인한다.
2. `git branch --show-current` 로 현재 브랜치(head)를 확인한다.
   - head와 base가 같으면 PR을 만들 수 없으므로, 사용자에게 알리고 종료한다.
3. `Skill(skill: "commit")` 실행.
4. `Skill(skill: "push")` 실행. push가 실패하거나 스킵되면(push할 커밋이 없고 원격에도 없으면) PR 생성 없이 이유를 보고하고 종료한다.
5. 이미 해당 head 브랜치로 열려 있는 PR이 있는지 확인한다: `gh pr view <head> --json url -q .url` (에러/빈 값이면 없는 것).
   - 이미 있으면 새로 만들지 않고 기존 PR URL만 보고하고 종료한다 (push로 이미 갱신됨).
6. 없으면 PR을 생성한다.
   - 제목: 직전 커밋 메시지(`git log -1 --pretty=%s`)를 기반으로 한 줄 요약. commit 커맨드가 이미 만든 `<prefix> <요약>` 형식을 그대로 재사용해도 된다.
   - 본문: `git log --oneline <base>..HEAD` 결과를 bullet list로 정리한 간단한 요약 (diff 인용 금지).
   - 실행: `gh pr create --base <base> --head <head> --title "<제목>" --body "<본문>"`
7. 생성된 PR URL을 사용자에게 한 줄로 보고한다.

## 주의사항
- head와 base가 동일한 브랜치를 가리키는 PR은 만들지 않는다.
- `gh pr create`에 `--base`로 존재하지 않는 브랜치를 넘기지 않는다 — 의심되면 먼저 `git branch -r` 등으로 가볍게 존재를 확인한다.
- PR 생성은 원격 저장소에 공개적으로 드러나는 동작이므로, base 브랜치가 `main`/`master`가 아닌 경우에도 그대로 진행하되, 6번 단계 실행 직전까지 실제 `gh pr create`는 호출하지 않는다 (준비 단계에서 미리 실행 금지).
- force push, 브랜치 삭제, 기존 PR을 닫거나 병합하는 동작은 하지 않는다. 이 커맨드는 PR 생성까지만 수행한다.
