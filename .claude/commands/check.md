---
description: 문법 검사(typecheck), lint, 테스트를 실행하고 결과를 한 줄로 보고합니다.
allowed-tools: Bash(npx tsc --noEmit:*), Bash(mypy:*), Bash(python -m mypy:*), Bash(python -m compileall:*), Bash(ruff check:*), Bash(flake8:*), Bash(npm run lint:*), Bash(pnpm run lint:*), Bash(yarn lint:*), Bash(npm test:*), Bash(npm run test:*), Bash(pnpm test:*), Bash(pnpm run test:*), Bash(yarn test:*), Bash(pytest:*), Bash(python -m pytest:*), Bash(npm run typecheck:*), Bash(pnpm run typecheck:*), Bash(yarn typecheck:*), Bash(npm run build:*), Bash(pnpm run build:*), Bash(yarn build:*)
---

인자를 받을 수 있습니다: `$ARGUMENTS`

## 인자가 있는 경우 (최우선)
`$ARGUMENTS`가 비어있지 않으면, 그 값을 **그대로 하나의 셸 명령어**로 실행한다.
(예: `/check npm run verify` → `npm run verify` 실행)
자동 감지 절차(아래)는 전부 건너뛴다. 결과 보고 방식은 동일하다.
(`allowed-tools`는 자동 감지 경로의 알려진 명령들만 사전 승인해두었으므로, 인자로 들어온 임의 명령은
자동 승인되지 않고 평소처럼 실행 전 승인을 거친다.)

## 인자가 없는 경우 — 자동 감지
아래 3단계를 순서대로 실행한다: **1) 문법 검사 → 2) lint → 3) test**.
각 단계는 프로젝트 설정 파일을 보고 명령어를 자동으로 판단한다.

### 명령어 감지 방법 (단계별)
- **문법 검사**: `tsconfig.json` → `npx tsc --noEmit`. TS가 아니면 `package.json`의
  `scripts.typecheck`/`scripts.build`. Python이면 mypy 설정(`mypy.ini`, pyproject의 `[tool.mypy]`) →
  `mypy .`, 없으면 `python -m compileall -q .`.
- **lint**: `package.json`의 `scripts.lint`. Python이면 `ruff`(pyproject `[tool.ruff]`, `.ruff.toml`) →
  `ruff check .`, 없으면 flake8 설정 → `flake8`.
- **test**: `package.json`의 `scripts.test`. Python이면 `pytest.ini`/`pyproject`의 pytest 설정/`tests/` 디렉토리 →
  `pytest -q`.
- JS/TS 계열 패키지 매니저는 lockfile로 판단한다: `pnpm-lock.yaml`→pnpm, `yarn.lock`→yarn, 그 외 npm.

### 명령어를 찾지 못한 단계가 있으면
그 단계는 실행하지 말고, 바로 아래와 같이 **한 줄로** 사용자에게 물어본 뒤 응답을 기다린다
(다음 단계로 임의로 넘어가거나 추측한 명령어를 실행하지 않는다):
`"<단계명> 명령어를 감지하지 못했습니다. 실행할 명령어를 알려주세요."`

## 토큰 절약 지침 (중요)
- 각 단계 실행 시 전체 stdout/stderr를 그대로 다 읽지 않는다. 종료 코드(exit code)만 우선 확인하고,
  실패한 경우에만 출력에서 **핵심 에러 줄 1~2개**만 뽑아본다.
- 성공한 단계의 로그는 전혀 인용하지 않는다.
- 테스트 프레임워크의 verbose 출력, 전체 lint 경고 목록 등을 길게 붙여넣지 않는다.

## 결과 보고
- **모든 단계 통과**: 다른 말 없이 정확히 한 줄만 출력한다 → `check pass`
- **어느 단계든 실패**: 아래 형식으로 **한 줄만** 보고한다 (여러 줄 로그 금지):
  `"<단계명> 실패: <핵심 에러/실패 사유 한 줄>"`
  - 예: `lint 실패: src/utils.ts:12 'x' is defined but never used`
  - 예: `test 실패: auth.test.ts › login 실패 (expected 200, got 401)`
  - 실패 시 이후 단계는 실행하지 않고 즉시 보고를 마친다.

## 주의사항
- 실패해도 코드를 임의로 수정하거나 재시도하지 않는다. 결과만 보고한다.
- 인자로 받은 명령어에 `rm`, `git push --force` 등 위험한 명령이 섞여 있으면 실행 전에 사용자에게 확인한다.
