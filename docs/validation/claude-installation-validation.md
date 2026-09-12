# Claude Code 플러그인 설치 검증

2026-09-12, `main`에 병합된 `.claude-plugin/` 매니페스트로 로컬 Claude Code 환경에 실제 설치·발견을 검증했다. 기준 커밋은 `818f15e`, 버전은 `0.1.0`이다.

## 매니페스트 구성

| 역할 | 경로 |
|---|---|
| 마켓플레이스 매니페스트 | `.claude-plugin/marketplace.json` (레포 루트) |
| 플러그인 매니페스트 | `report_writing/.claude-plugin/plugin.json` (코덱스의 `.codex-plugin/plugin.json`과 같은 폴더에 나란히 위치) |
| 스킬 자동 탐색 | `report_writing/skills/` — Claude 관례상 `plugin.json` 옆의 `skills/`를 별도 필드 선언 없이 자동 탐색 |

마켓플레이스 루트(레포 루트)와 플러그인 루트(`report_writing/`)가 다르므로 `marketplace.json`의 `source`는 `"./report_writing"`으로 지정했다.

## 설치와 확인

```bash
claude plugin validate .                # 마켓플레이스 매니페스트 검사
claude plugin validate ./report_writing # 플러그인 매니페스트 검사
claude plugin marketplace add "<repo 절대경로>"
claude plugin install report_writing@report-writing-skills
claude plugin list
claude plugin details report_writing@report-writing-skills
```

`validate`는 통과했으나 경고 1개가 있다: 플러그인 이름 `report_writing`이 kebab-case가 아니다. Claude Code 자체는 이 이름을 허용하지만 claude.ai 마켓플레이스 동기화는 kebab-case를 요구한다. 코덱스 쪽 스킬 이름(`report_writing:brainstorm` 등)과 동일하게 유지하기 위해 의도적으로 남긴 경고다.

`marketplace add`는 상대 경로(`.`)를 거부했고 절대 경로를 요구했다 — `owner/repo`, `https://...`, `./path` 중 `./path` 형태가 필요했다. 절대 경로로 재시도해 성공했다.

설치 결과:

| 확인 항목 | 결과 |
|---|---|
| `claude plugin list` | `report_writing@report-writing-skills`, version `0.1.0`, scope `user`, status `enabled` |
| `claude plugin details` | Skills (4): `brainstorm`, `develop`, `spec`, `writing-plan`. Agents/Hooks/MCP/LSP 모두 0 |
| 항상 로드되는 토큰 비용 | 세션당 ~190 tok (스킬당 ~50 tok) |
| 호출 시 비용 | brainstorm ~610, spec ~810, writing-plan ~930, develop ~1.1k |

이 검증은 실제 설치·활성화와 플러그인 매니페스트 기준 컴포넌트 인벤토리 확인이다. 이미 실행 중이던 세션에서는 `/reload-plugins`를 실행해야 `Skill` 도구 목록에 네 이름이 반영됐다 (`report_writing:brainstorm`, `report_writing:spec`, `report_writing:writing-plan`, `report_writing:develop`). 새 세션은 시작 시 자동으로 반영되므로 재로드가 필요 없다.

## 사용과 이후 갱신

새 Claude Code 대화를 열고 `report_writing:brainstorm` 등 이름을 명시해 호출한다. 자동 호출은 SKILL.md의 `description` 문구 자체로 막혀 있으므로(코덱스의 `allow_implicit_invocation: false`와 동일한 효과) 이름을 직접 불러야 한다.

개발 소스(`report_writing/`)를 수정한 뒤에는 `claude plugin marketplace update report-writing-skills`로 마켓플레이스를 갱신하고 `claude plugin update report_writing` 또는 재설치로 반영한다. 이 저장소를 다른 위치에 클론하면 `marketplace add` 경로를 다시 등록해야 한다.
