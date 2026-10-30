# enter-flow 설정 찾기

모든 enter-flow 스킬이 작업을 시작할 때 이 순서로 값을 정하고, 정한 값과 근거를 한 줄로 알린다.
(예: `문서 위치: .enter-flow/ (기본값) · 기준 브랜치: dev (CLAUDE.md) · 커밋 형식: .claude/skills/commit`)

## 문서 위치 (DOCS)
1. 프로젝트 CLAUDE.md(또는 CLAUDE.local.md)의 `enter-flow 문서 위치: <경로>`
2. 없으면 `<레포 루트>/.enter-flow/`
   처음 쓸 때 `.enter-flow/.gitignore`(내용 `*`)를 만든다. 팀 파일(.gitignore 등)은 건드리지 않는다.
- 아래 폴더를 쓴다: `DOCS/designs/`, `DOCS/plans/`, `DOCS/reviews/`
- cd 하지 않고 전체 경로로 다룬다 (세션 폴더가 바뀌면 레포 스킬이 사라진다).

## 기준 브랜치 (BASE)
1. CLAUDE.md의 `enter-flow 기준 브랜치: <이름>`
2. 원격 기본 브랜치: `git symbolic-ref --short refs/remotes/origin/HEAD` (앞의 `origin/` 제외)
3. `main`, 없으면 `master`
- 구현하지 않는 브랜치: BASE, main, master. 여기면 멈추고 알린다.

## 규칙 찾기 (RULES)
1. 프로젝트 CLAUDE.md, CLAUDE.local.md
2. `.claude/rules/` (바뀔 파일 경로에 맞는 것)
3. CONTRIBUTING.md, `docs/` 안에서 파일 이름에 convention·style·guide가 들어간 문서
4. 하나도 없으면 "규칙 문서 없음 — 기존 코드 패턴을 따름"이라고 밝힌다
- 근거 순서: 규칙 → 기준 브랜치에 머지된 코드 → 이슈 → 열린 PR(참고만). 부딪히면 앞의 것을 따른다.
  (열린 PR은 아직 합의되지 않은 코드라 기준으로 삼으면 규칙과 어긋나기 쉽다)

## 커밋 형식 (COMMIT)
1. 레포의 commit 스킬 (`.claude/skills/commit/SKILL.md`) — 있으면 Skill 도구로 쓴다
2. 규칙 문서(위 RULES)의 커밋 규칙
3. 최근 커밋 20개(`git log -20 --format=%s%n%b`)의 형식을 따라 한다
4. 아무것도 없으면 Conventional Commits (`feat: ...`, `fix: ...`)
- AI 표기(Co-Authored-By 등)도 같은 순서로 찾은 규칙을 따른다. 규칙이 없으면 Claude Code 기본 설정을 따른다.
- 무엇을 근거로 골랐는지 장부·보고에 한 줄 남긴다.

## PR 형식 (PR)
1. 레포의 pr 스킬 (`.claude/skills/pr/SKILL.md`)
2. `.github/pull_request_template.md` (대소문자 무관)의 섹션
3. 없으면: 개요 / 작업 내용 / 참고 사항 / 관련 이슈 / 완료 조건

## 언어
- 사용자에게 하는 말과 문서는 사용자가 쓰는 언어로 쓴다.
