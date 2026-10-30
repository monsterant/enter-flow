# enter-flow

작업을 시작할 때 `/enter:autoEnterEnter` 하나만 부르면, 작업 종류에 맞춰
**질문으로 정리 → 설계 → 구현 계획 → 구현·리뷰**를 정해진 순서대로 진행하는 Claude Code 플러그인입니다.
"이거 이거 하자"고 할 때마다 스킬이나 점검이 하나씩 빠지는 것을 막으려고 만들었습니다.

## 들어 있는 스킬

| 명령 | 하는 일 |
|---|---|
| `/enter:autoEnterEnter` | 작업 시작 메뉴. 1 설계부터 / 2 코드 작성 / 3 코드 수정 (설계 없이) / 4 PR 마무리 / 5 코드 재수정 / 6 코드 리뷰 |
| `/enter:grill` | 무엇을 할지(용어·조건·결정)를 질문으로 하나씩 맞춥니다 |
| `/enter:design` | 어떻게 짤지 9개 항목(흐름·통신·테이블·상태·API·중복·실패·대안·1차/2차)으로 설계합니다 |
| `/enter:plan-it` | 설계를 구현 단계(파일·시그니처·테스트·커밋 경계)로 옮깁니다 |
| `/enter:run-plan` | 계획을 에이전트로 구현하고 단계 리뷰·3관점 리뷰를 돌립니다. 리뷰 전용 모드 포함 |

모든 스킬은 **직접 부를 때만** 실행됩니다. 보통은 `/enter:autoEnterEnter`로 시작하면 나머지를 순서대로 이어 줍니다.

## 설치

Claude Code 세션에서:

```
/plugin marketplace add monsterant/enter-flow
/plugin install enter@enter-flow
```

또는 셸에서 `claude plugin marketplace add monsterant/enter-flow` → `claude plugin install enter@enter-flow`.

## 설정 (선택)

아무 설정 없이 동작합니다. 바꾸고 싶으면 프로젝트 `CLAUDE.md`에 아래 줄을 넣습니다.

```
enter-flow 문서 위치: docs/work/
enter-flow 기준 브랜치: dev
```

| 값 | 넣지 않았을 때 |
|---|---|
| 문서 위치 (설계·계획·리뷰 문서) | `<레포>/.enter-flow/` — git에 잡히지 않게 자동 처리 |
| 기준 브랜치 | 원격 기본 브랜치, 없으면 `main`/`master` |
| 커밋 형식 | 레포의 commit 스킬 → 규칙 문서 → 최근 커밋 20개 → Conventional Commits 순으로 찾음 |
| 규칙 | CLAUDE.md → `.claude/rules/` → CONTRIBUTING·docs의 컨벤션 문서 순으로 찾음 |

자세한 순서는 `skills/autoEnterEnter/references/settings.md`에 있습니다.

## 필요한 것

- git, GitHub CLI(`gh`) — 이슈·PR을 읽을 때
- bash — `run-plan`의 스크립트 (Windows는 Git Bash)

## 안전장치

- push, PR 생성, GitHub 쓰기는 승인 없이 하지 않습니다.
- 자동 모드에서도 로컬 코드 수정·테스트·로컬 커밋까지만 합니다.
- 기준 브랜치·main·master에서는 구현하지 않고 멈춥니다.

## 출처

`run-plan`은 [obra/superpowers](https://github.com/obra/superpowers)의 subagent-driven-development(MIT)를 고친 것입니다. 자세한 내용은 `skills/run-plan/ORIGIN.md`.

## 라이선스

MIT
