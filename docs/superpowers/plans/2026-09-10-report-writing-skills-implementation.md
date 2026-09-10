# Report Writing Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 자료·초안·작성 조건에서 시작하여 사용자의 검토와 승인을 거쳐 HTML 보고서를 작성하는 `report_writing` 네 단계 스킬을 구현한다.

**Architecture:** 개발 저장소 안의 `report_writing/`를 플러그인 루트로 사용한다. 네 개의 짧은 단계 스킬은 공통 진행 규칙·산출물 계약을 참조하고, 집필 단계만 상세 작성 기준과 HTML 자산을 읽는다. 단일 에이전트가 보고서를 작성하며 승인 상태는 문서별 작업 기록에 보존한다.

**Tech Stack:** Codex plugin manifest(JSON), SKILL.md(Markdown/YAML), agents/openai.yaml, 공통 Markdown 참조, HTML/CSS. 검증은 공식 Python 검사기와 독립 실행 시나리오, 로컬 브라우저를 사용한다.

**Spec:** [검토 완료된 구현 기준 스펙](../specs/2026-09-09-report-writing-skill-design.md), 기준 커밋 `ed61bd7`.

## Global Constraints

아래는 기준 스펙의 공통 요구사항이다. 구현 작업 전체에 적용한다.

- “하나의 보고서 작성 흐름을 `report_writing` 아래의 네 단계 스킬로 제공하며, 각 단계는 단일 에이전트가 수행한다.”
- “사용자의 명시적 호출로 진입하고 AGENTS.md 자동 실행은 넣지 않는다.”
- “별도 피드백 스킬과 서브에이전트 방식은 후속 작업으로 보류한다.”
- 호출: `report_writing:brainstorm`, `report_writing:spec`, `report_writing:writing-plan`, `report_writing:develop`.
- “최종 보고서는 브라우저에서 읽을 수 있는 `report.html`로 작성한다.”
- “사용자가 경로를 지정하면 그 경로를 따른다.” “기존 작업을 재개할 때는 확인된 기존 경로를 유지한다.”
- 기본 저장: `reports/<report-id>/report.html`, `work/brief.md`, `work/materials.md`, `work/spec.md`, `work/writing-plan.md`, `work/state.md`. 그림이 필요하면 `work/figures.md`.
- “피규어: 생성하지 않는다.” 플랜은 그림의 핵심, 라이팅은 빈자리·실제 설명·캡션·각주와 연결된 의도·내용·간단한 스케치 구성 메모를 만든다.
- “섹션별·문단별 사용자 승인이나 별도 그룹 6·7 승인 단계를 추가하지 않는다.”
- “자체 체크리스트 충족 + 단계 전체 내용 승인 + 구조 승인 + 진행 승인”을 충족하면 다음 단계로 넘어간다. 최종 라이팅은 원고 완료 승인을 기록한다.
- “이미 승인한 범위와 구체적인 사용자 지시의 승인 효력을 인정하고 같은 사항을 반복해서 묻지 않는다.”
- “SKILL.md만으로 시스템 수준의 실행 차단이 보장되는 것은 아니다.”
- “문장 수·문단 길이·불렛 개수의 고정 수치를 통과 기준으로 삼지 않는다.”

이 계획은 **스킬을 만드는 개발 계획**이다. 런타임 `writing-plan`의 보고서 설명 계획과 구별한다. 설계 세부안의 재승인 단계는 없다. 이 구현 계획 검토 후 실제 파일을 구현·검증한다.

---

## 1. 파일 구성과 구현 경계

전체 흐름은 하나의 계획으로 구현한다. 아래 파일은 이 계획 실행 시 생성하며, 현재 계획 작성 작업에서는 생성하지 않는다.

| 파일 | 책임 |
|---|---|
| `report_writing/.codex-plugin/plugin.json` | 네임스페이스와 스킬 탐색 경로 |
| `report_writing/skills/brainstorm/SKILL.md` | 자료·미세한 의도 탐색, 짧은 작성안 |
| `report_writing/skills/spec/SKILL.md` | 섹션별 제목·핵심·참조의 선별 |
| `report_writing/skills/writing-plan/SKILL.md` | 실제 설명과 설명 순서로 확장 |
| `report_writing/skills/develop/SKILL.md` | 원고 집필·편집·HTML 확인 |
| 위 네 폴더 각각의 `agents/openai.yaml` | 수동 호출 정책 |
| `report_writing/references/workflow.md` | 공통 승인·변경·체크·진입·재개 |
| `report_writing/references/artifacts.md` | 문서별 자료·스펙·플랜·상태 기록의 계약 |
| `report_writing/references/writing-criteria.md` | G6-1~G7-4와 표현 방식·점검 |
| `report_writing/references/connected-example.md` | 실제 설명의 상세도와 섹션·소제목 경계를 보여주는 자족적인 적용 예시 |
| `report_writing/skills/develop/assets/report.html` | 문서 구조·기본 스타일·출처·빈자리의 HTML 예시 자산 |
| `tests/fixtures/kiosk/materials.md` | 별도 주제의 합성 원자료. 평가 대상 에이전트에게 제공 |
| `tests/scenarios.md` | 실행 프롬프트·시작 상태·후속 사용자 응답 |
| `tests/rubric.md` | 평가자만 읽는 행동·산출물 판정 기준 |
| `docs/validation/implementation-validation.md` | 모델·조건·실제 산출물·실패·수정·재검증 기록 |
| `README.md`, `.gitignore`, 기준 스펙 §6 | 사용법·개발 상태·임시 평가 산출물 제외 |

스킬의 참조는 `../../references/workflow.md`처럼 **SKILL.md가 있는 폴더 기준**으로 연결한다. 네 스킬과 공통 참조를 플러그인 전체로 함께 옮긴다. 개발 문서·상위 보고서 프로젝트를 설치 후 실행의 필수 참조로 사용하지 않는다. 저장소 이름과 기존 설계 문서의 위치는 유지한다. 루트의 빈 `skills/.gitkeep`은 Task 1에서 제거하고 README의 스킬 경로를 실제 패키지 경로로 바꾼다.

설치·마켓플레이스 등록·배포는 이 계획의 완료 조건에 포함하지 않는다. 로컬 패키지의 구성 검증과 직접 파일을 읽힌 행동 검증까지 수행하고, 실제 앱에서 호출이 발견되는지는 설치 후 검증 항목으로 구분한다.

### 공통 참조를 읽는 시점

| 참조 | 읽을 조건과 범위 |
|---|---|
| `workflow.md` | 매 진입에 entry·gates·checks, 수정이 생기면 changes, 기존 초안·재개이면 resuming |
| `artifacts.md` | 최초 진입에 identity·state, 각 단계에서 해당 산출물 절, 입력 자료가 바뀌면 materials |
| `writing-criteria.md` | develop의 집필·자체 점검에서 전체 적용. 이전 단계에서는 해당 단계 지침에 정한 명확성 기준 사용 |
| `connected-example.md` | 목표→실제 설명의 상세도, 소제목의 소속, 표현 보완과 실질 변경의 경계가 모호할 때 해당 예시 확인 |
| `assets/report.html` | HTML 작성 시 구조·스타일의 출발점으로 사용. 예시의 내용·문단 수·표 배치는 문서 조건에 맞게 교체 |

공통 파일을 또 다른 자동 호출 스킬로 만들지 않는다. 다음 단계로의 정상 진행은 사용자의 기존 진행 승인과 필수 입력을 확인한 뒤 해당 단계의 SKILL.md를 읽어 이어간다. 단계를 직접 호출했는데 입력이 부족하면 공통 진입 규칙으로 필요한 준비를 하되, 파일 존재만으로 승인된 것으로 간주하지 않는다.

## 2. 단계 사이 데이터 계약

`artifacts.md`에 아래 필드와 의미를 명시한다. Markdown 서식은 읽기 쉽게 조정할 수 있지만 필드의 의미와 연결을 보존한다. 별도 JSON 상태 엔진이나 파서는 만들지 않는다.

### 2.1 식별과 버전

- 문서 루트는 사용자 지정 → 재개 중인 기존 루트 → `reports/<report-id>/` 순으로 선택한다. 새 기본 ID는 주제를 나타내는 짧은 이름으로 정하고, 같은 이름의 기존 작업을 확인한다. 무관한 작업이면 구별 가능한 접미사를 붙인다.
- 각 산출물 머리에 문서명·버전·사용한 선행 버전을 적는다. 새 문서의 초기 버전은 `v0.1`, 내용·구조가 바뀌면 식별 가능한 다음 버전을 사용한다. 이미 쓰는 버전 방식이 있으면 유지한다.
- `S` = Section, `P` = Plan, `M` = Material, `U` = User. 처음 한 번 범례를 둔다. P는 원고 한 문단과 동일하지 않다.
- 의미상 섹션과 표시용 소제목은 기준 스펙 §2.1의 정의를 사용한다. 새로 목표·범위를 배정한 하위 섹션은 별도 S이고, 표시용 소제목은 기존 S에 소속된다.
- 자료 ID·플랜 ID는 수정 중에도 대응을 추적할 수 있게 유지한다. 원고에는 작업용 ID 대신 독자용 제목·출처를 쓴다.

### 2.2 산출물별 내용

| 산출물 | 필수 내용과 표시 |
|---|---|
| brief | 목적·독자·문서 유형·필수 형식·표현 방식·강조점·범위·제외 범위·최신 지시 위치. 큰 구성과 앞뒤 연결을 담은 짧은 작성안. 핵심 선택에 영향을 줄 미해결 질문 |
| materials | ID, 원자료/사용자 의도 구분, 원문 위치와 절·항목, 내용 안내 또는 참조 이유, 확인 상태. 선택 이유·미채택 자료·조건·제외 범위·추정한 의도는 참고 기록으로 보존 |
| spec | 승인/후보 상태와 선행 brief 버전. S별 제목·하나의 핵심·짧은 우선 참조. 성립 조건은 핵심 옆에서 보이게 하고 소속·범위가 모호할 때 짧게 명시. 상세 설명·점검은 work 기록에 연결 |
| writing-plan | 선행 spec·brief 버전. S별 목표·범위 연결 아래 P를 실제 설명 순서로 배치. P의 소속·역할, 실제 설명, 근거 M/U, 필요한 조건, 그림의 중심 메시지. 배치 지시는 실제 설명과 구분 |
| state | 대상·조건·각 파일 위치/버전, 단계별 승인 표, 체크 표, S/P별 작성·점검 상태, 변경 묶음과 영향 범위, 미해결 사항·다음 작업 |
| report.html | 독자가 읽을 원고, 필요한 표·출처·각주·그림 빈자리와 캡션. 필요한 스타일 포함. 내부 승인/점검 로그는 work에 둠 |
| figures | 그림별 연결 위치, 설명할 것·의도·내용·간단한 스케치 구성, 실제 캡션·각주와 본문 위치의 연결. 실제 그림은 생성하지 않음 |

**자료 계약:** 우선 참조는 위치와 참고할 내용/이유를 함께 갖는다. 포인터의 요약만으로 사실을 추측하지 않는다. 플랜은 연결된 원문을 먼저 읽고 필요하면 더 넓은 자료 풀로 확장한다. 포인터를 닫힌 목록이나 필수 수록 목록으로 만들지 않는다. 원문 접근을 짧은 요약으로 대체하지 않는다.

**스펙 예시 — 목표를 압축하는 수준:**

> S8 · 증류 경로를 선택·조합하는 통합 증류기
>
> 핵심: 필요한 증류 방식을 선택해 함께 사용할 통합 증류기의 구현 방식을 설명한다.
>
> 우선 참조: U1 — 선택 기능을 중심에 둘 의도. M2 — 모듈 선별과 적재 순서. M3 — 같은 step의 이미지 인코딩 공유.

**플랜 예시 — 실제 설명을 담는 수준:**

> S8 / P2 · 메모리 관리
>
> 실제 설명: 체크포인트를 CPU에 읽은 뒤 선택한 증류 경로에 필요한 티처 모듈을 남겨 GPU로 옮긴다. LM만 사용할 때는 이미지 인코더와 텍스트 디코더가 필요하고, ITM을 함께 사용하면 텍스트 인코더와 ITM 헤드도 필요하다.
>
> 근거: M2. 배치: S8의 표시용 소제목 아래. 핵심 기능 소개 다음에 둔다.

최초 플랜에서 이런 하위 설명과 포인터를 선택하는 것은 승인된 목표 안의 확장이다. 승인된 플랜이 생긴 이후 필수 설명의 의미를 바꾸는 경우에는 변경 규칙을 적용한다.

### 2.3 승인·체크·진행 기록

`state.md`의 표는 다음 열을 갖는다. 아래는 **필드 정의**이며, 빈 파일에 승인 완료 예시를 기본값으로 채우지 않는다.

| 표 | 열 |
|---|---|
| 산출물 | 단계, 경로, 버전, 선행 버전, 작성 상태 |
| 승인 | 단계, 버전, 범위, 내용, 구조, 진행/완료, 사용자 발언·기록 위치 |
| 자체 체크 | 단계, S/P 또는 전체, 기준 ID, 확인 조건, 상태, 확인 위치·근거, 필요한 수정 |
| 진행 | S/P, 원고 위치, 작성됨, 점검됨, 승인 범위와 대응, 남은 일 |
| 변경 묶음 | 식별명, 기존 결정, 구체적 수정안, 이유, 표현/플랜/스펙 구분, 영향 위치, 승인 상태 |
| 다음 작업 | 미해결 질문·자료, 의존 범위, 계속 가능한 독립 작업, 다음 위치 |

- 승인 상태는 `미승인 / 부분 승인 / 승인`이며, 현재 사용자의 구체적 지시도 근거로 기록한다.
- 체크 상태는 `미확인 / 미충족 / 충족 / 해당 없음`이다. 해당 없음은 조건부 기준에만 적용하고 이유를 남긴다.
- 기존 승인된 미변경 범위는 새 버전에 승계됐음을 기록한다. 변경 범위의 옛 체크를 현재 충족으로 복사하지 않는다.
- 사용자에게 제시한 파일·버전·범위를 기준으로 승인한다. 자료 풀의 내부 후보를 사용자 결정으로 바꾸지 않는다.
- 사용자 한 답변이 내용·구조·진행을 함께 승인하면 각 항목에 같은 근거를 연결한다. 항목 수만큼 다시 묻지 않는다.
- 승인 대기 중에는 구체적 변경안과 영향받지 않는 부분을 준비한다. 준비한 제안을 승인된 원고로 확정하지 않는다.

### 2.4 기준 ID와 적용 단위

이 ID는 기록용 이름이며 새로운 사용자 승인 단계를 뜻하지 않는다.

| 단계 | 필수 체크 |
|---|---|
| brainstorm | B1 목적·독자·형식·강조점·범위 확인, B2 큰 흐름과 자료 위치 파악, B3 핵심 선택을 바꿀 미해결 질문 해소 |
| spec | S-G3 제목과 범위 대응, S-G4 중심 목표 하나, S-G5 설명 가능성과 근거·조건 연결. 모든 S 및 목표를 묶는 상위 S에 적용 |
| writing-plan | P-G3 제목·소속 범위, P-G4 같은 핵심, P-G5 실제 설명·필수 연결·문장별 역할. 모든 S/P와 그 안의 설명 문장에 적용 |
| develop | D-G345 승인된 제목·핵심·설명·근거·조건 보존 + G6-1~G7-4. 모든 S의 소제목·본문·연결에 적용 |
| 모든 단계 | ALL-G12 작성 조건 일치·전체 누락·중복·순서·흐름. 단계 산출물 전체에서 확인 |

G6-1~G7-4는 스펙 §4.2의 지침과 통과 조건을 함께 유지한다. 특정 표·그림이 없더라도 설명 관계·용어·핵심 보존 기준은 필수다.

## 3. 검증 방식과 공통 실행 준비

### 3.1 검증을 나누는 이유

공식 검사기는 manifest/frontmatter/정책 형식을 확인한다. 글의 충분함·가독성·승인 범위 판단은 실제 요청을 수행한 결과로 확인한다. 문자열에 ‘충족’이 있는지를 세는 테스트로 행동 품질을 판정하지 않는다.

Task 1~4의 중간 완료는 해당 단계의 구현·검증을 뜻한다. 다음 단계 파일이 모두 갖춰진 전체 패키지의 사용 가능 판정은 Task 5에서 한다.

Task 1~4에서 각각 **스킬 없는 기준 실행 → 해당 스킬 구현 → 같은 조건의 독립 실행 → 관찰된 실패 수정**을 진행한다. 이는 개발 중 평가이며 런타임의 서브에이전트 피드백 기능을 구현하는 것이 아니다.

- 평가 에이전트는 새 문맥(`fork_turns="none"`)으로 시작한다. 실행에 필요한 요청·최소 자료·허용 출력 폴더만 준다.
- 기준 실행에는 이 계획·설계 스펙·정답·루브릭·대상 스킬을 주지 않는다. 스킬 실행에는 해당 SKILL.md 경로와 공통 참조가 들어 있는 패키지를 추가한다.
- 평가 모델은 실행 세션의 주 모델을 기본으로 사용하고 실제 모델/추론 설정을 기록한다. 두 비교 실행은 같은 조건을 사용한다. 별도 모델 지정을 사용자가 주면 그것을 따른다.
- 기존 대화가 있는 재개 시나리오는 `tests/scenarios.md`의 대화 발췌를 동일하게 제공한다. 시뮬레이션된 승인은 평가 폴더 안에서만 유효하다.
- 기준 실행이 이미 통과하면 성공으로 기록한다. 실패를 만들어내기 위해 사실을 바꾸거나 불필요한 금지 문구를 늘리지 않는다. 합의된 계약의 구현과 실제 실패에 필요한 지침을 구별한다.
- 특정 행동 유도 문구를 비교·개선할 때는 무지침 대조군을 포함해 각 변형을 새 문맥으로 5회 이상 확인한다. 고정 문구/문단 수가 아니라 실제 해석·결정의 일관성을 판정한다. 모든 시나리오를 이유 없이 반복하지 않는다.
- 실패는 원문 응답·파일 위치·위반 기준을 남기고 영향받는 부분을 수정한다. 수정 후 관련 시나리오를 다시 실행한다.

평가 결과는 `docs/validation/implementation-validation.md`에 기준 실행/스킬 실행, 모델·조건, 결과 위치, 실제 행동, 판정, 수정, 재검증을 기록한다. 생성물은 `.work/validation/<case>/<run>/`에 두고 `.work/`를 Git에서 제외한다. 핵심 전후 발췌와 판정은 검증 문서에 남겨 재현 근거를 보존한다.

### 3.2 실제 검증 환경

2026-09-10 로컬에서 Python과 Edge/Chrome 경로를 확인했다. Python의 PyYAML은 아직 설치되어 있지 않다. 아래 준비는 **구현 실행 시** 수행한다.

```powershell
$reportPython = 'C:/Users/minwoo/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/python.exe'
if (-not (Test-Path -LiteralPath './.venv/Scripts/python.exe')) {
    & $reportPython -B -X utf8 -m venv .venv
    if ($LASTEXITCODE -ne 0) { throw 'Virtual environment creation failed.' }
}
& ./.venv/Scripts/python.exe -B -X utf8 -m pip install PyYAML
if ($LASTEXITCODE -ne 0) { throw 'PyYAML installation failed.' }
```

이는 개발 검증용 의존성이다. 보고서 작성 스킬의 런타임 필수 의존성으로 선언하지 않는다. 다운로드가 제한되면 환경의 승인 절차를 따르고 설치·검사가 되지 않은 항목을 통과로 기록하지 않는다.

공식 검사기 위치:

- `C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py`
- `C:/Users/minwoo/.codex-lab/skills/.system/plugin-creator/scripts/validate_plugin.py`

각 Task에서 해당 스킬을 검사하고 Task 5에서 플러그인 전체를 검사한다. UTF-8로 읽도록 `-X utf8`를 사용한다.

## 4. 구현 작업

### Task 1: 공통 인계 계약과 brainstorm 구현

**Requirements:** 스펙 §1~§3, §4.3의 표현 방식 유지, 이 계획 §2.

**Files:**
- Create: `tests/fixtures/kiosk/materials.md`, `tests/scenarios.md`, `tests/rubric.md`, `docs/validation/implementation-validation.md`
- Create: `report_writing/.codex-plugin/plugin.json`
- Create: `report_writing/references/workflow.md`, `report_writing/references/artifacts.md`
- Create: `report_writing/skills/brainstorm/SKILL.md`, `report_writing/skills/brainstorm/agents/openai.yaml`
- Modify: `.gitignore`, `README.md`; Remove: `skills/.gitkeep`

**Interfaces:**
- Consumes: 현재 요청, 제공 자료/초안, 기존 work 파일과 확인된 승인.
- Produces: §2 계약의 materials·brief·state. 다음 단계는 이 셋과 사용자 승인을 입력으로 사용한다.
- Shared resource anchors: workflow의 `entry`, `gates`, `checks`, `changes`, `resuming`; artifacts의 `identity`, `brief`, `materials`, `spec`, `writing-plan`, `state`, `figures`.

- [ ] **Step 1: 평가 자료와 독립 요청을 작성한다.**

`tests/fixtures/kiosk/materials.md`에는 다음 데이터를 그대로 넣는다. **검증용 합성 자료이며 실제 실험 결과가 아님**을 머리에 표시한다.

```markdown
# 회의실 예약 키오스크 평가 · 합성 자료
## M1 · 구현과 선택 이유
기존 A는 방 선택, 시간 선택, 최종 확인의 세 화면이다.
B는 방과 시간을 한 화면에서 선택하고 다음 화면에서 최종 확인한다.
개발 중 최종 확인을 없앤 안도 시도했으나, 선택을 다시 확인하기 어렵다는
내부 검토 의견 때문에 최종 확인을 유지했다. 이 내부 검토에는 정량 측정이 없다.
## M2 · 관측 기록
성인 참여자 12명이 동일한 예약 과제를 A, B 순으로 한 번씩 수행했다.
완료 시간 중앙값은 A 84초, B 63초였다. 성공은 A 10/12명, B 11/12명이었다.
순서를 바꾸거나 무작위 배정하지 않았으며 별도 집단 실험도 하지 않았다.
성공은 지정된 방과 시간으로 최종 예약을 마친 경우를 뜻한다.
## M3 · 후속 계획
순서를 균형 있게 배정한 추가 평가를 계획했다. 아직 수행하지 않았다.
## M4 · 다른 재료
색상 테마 세 안과 아이콘 변경 이력이 있다. 예약 흐름 및 이번 측정과의
관계는 확인하지 않았다.
```

`tests/scenarios.md`의 B-1 요청:
“이 자료로 팀의 개발 결과보고서를 준비하자. 독자는 제품 책임자다. 가독성을 중시하고 피규어는 만들지 말자. 기능 변경과 성능 중 무엇을 중심에 둘지는 아직 고민 중이다. 우선 글감과 짧은 작성안을 잡아줘.”

후속 응답:
“중심은 방과 시간을 함께 선택하도록 흐름을 바꾼 이유와 구현이야. 관측 결과는 그 뒤에 제한을 포함해 보여주고, 추가 평가는 계획으로 남겨.”

평가자는 brief에 의도·관측·계획이 구분되고 전체 흐름이 형성되는지, 최초 불확실성을 확인하는지, 아직 스펙/원고를 승인된 결과로 밀어붙이지 않는지 본다. M4를 미채택해도 자료 접근은 유지해야 한다.

- [ ] **Step 2: B-1을 스킬 없이 실행하고 원문과 판정을 기록한다.**

평가 대상에는 §3.1 조건과 Step 1 자료·요청만 준다. 질문이 나오면 정해진 후속 응답을 전달한다. 실제 오류를 `tests/rubric.md`의 B1~B3·ALL-G12에 대응한다. 단순히 질문 수가 많거나 적다는 이유만으로 판정하지 않는다.

- [ ] **Step 3: 최소 manifest와 수동 호출 정책을 작성한다.**

`report_writing/.codex-plugin/plugin.json`:

```json
{
  "name": "report_writing",
  "version": "0.1.0",
  "description": "자료와 작성 조건에서 단계별 검토를 거쳐 HTML 보고서를 작성하는 스킬",
  "author": { "name": "ganadi" },
  "skills": "./skills/",
  "interface": {
    "displayName": "Report Writing",
    "shortDescription": "Write reports through four reviewed stages",
    "longDescription": "Collect materials, define a report spec, expand a writing plan, and develop an HTML report.",
    "developerName": "ganadi",
    "category": "Productivity",
    "capabilities": ["Read", "Write"],
    "defaultPrompt": ["Use $report_writing:brainstorm to start a report."]
  }
}
```

각 단계의 `agents/openai.yaml`에는 아래 구조를 사용한다. 공식 플러그인 검사기가 요구하는 두 UI 필드와 수동 호출 정책을 넣고, 아이콘·색상·스킬별 기본 프롬프트는 추가하지 않는다.

```yaml
interface:
  display_name: "Report Writing Brainstorm"
  short_description: "Gather report materials, intent, and writing conditions"
policy:
  allow_implicit_invocation: false
```

네 단계의 UI 값은 다음과 같다.

| 단계 | display_name | short_description |
|---|---|---|
| brainstorm | Report Writing Brainstorm | Gather report materials, intent, and writing conditions |
| spec | Report Writing Spec | Select section goals and source pointers for a report |
| writing-plan | Report Writing Plan | Expand report section goals into grounded explanations |
| develop | Report Writing Develop | Write a readable HTML report from an approved plan |

사용자가 지정한 Git 이름 `ganadi`를 필수 author/developer 표시에도 사용한다. 연락처·외부 URL은 추가하지 않는다. manifest의 기본 프롬프트는 UI의 시작 문구이며 자동 실행 설정이 아니다.

플러그인 이름의 밑줄은 유지한다. 스킬 폴더와 frontmatter의 name에는 `brainstorm`, `spec`, `writing-plan`, `develop`만 쓴다. 네임스페이스는 manifest가 제공한다.

- [ ] **Step 4: 공통 규칙을 출처에 대응해 옮긴다.**

`workflow.md`에는 다음 대응으로 **현재 규칙과 예외를 함께** 옮긴다. 과거 검토 상태·개발 과정 서술은 실행 규칙으로 옮기지 않는다. 기존 초안의 첫 묶음 검토 조건이 충족되면 brainstorm·spec·writing-plan의 SKILL.md를 읽어 각 단계의 후보를 만들고 자체 점검한 후 묶음 전체를 제시한다. 이 후보 준비 중에는 단계별 중간 승인 요청을 추가하지 않는다. 신규 문서의 정상 진행과 이미 승인된 자료에서 재개하는 경우를 이 예외와 구별한다.

| 대상 절과 명시적 anchor | 원문 범위 | 반드시 함께 둘 의미 |
|---|---|---|
| entry | 스펙 §1.1, §3.6의 진입 기준·표 | 호출명만으로 승인 추정 금지, 선행 상태로 진입 판단 |
| gates | §3.1~§3.2 | 단계 전체 검토, 4개 통과 요소, 복합 승인·기존 승인, 내부 후보의 경계 |
| checks | §3.3~§3.4 | 조건·범위·상태·근거, 모든 섹션과 전체, 필수/조건부 구분 |
| changes | §3.5 전체 | 세 수준 표와 부연, 구체적 변경 묶음·의존 범위·대기 중 가능한 일 |
| resuming | §3.6의 첫 묶음 검토와 최소 기록 | 초안 묶음 검토의 적용 조건, 일부 승인 보존, 현재 사용자 지시의 효력 |

각 절 앞에 `<a id="entry"></a>` 형태의 명시적 anchor를 둔다. 실행용 문서의 절 참조는 같은 파일의 anchor 또는 `artifacts.md` 해당 anchor로 다시 연결한다. 규칙의 원출처를 찾기 위해 개발 스펙 전체를 다시 읽도록 만들지 않는다.

`artifacts.md`에는 이 계획 §2의 필드·상태·ID·예시와 스펙 §2.1~§2.2를 통합한다. §2.4의 체크 ID를 해당 산출물 절에 배치한다. 문서별 파일과 상태의 계약은 여기에서만 정의한다.

- [ ] **Step 5: brainstorm의 본문을 작성한다.**

```markdown
---
name: brainstorm
description: Use when the user explicitly requests report_writing:brainstorm for report materials, intent, or an initial writing brief.
---

# 보고서 글감 브레인스토밍

자료와 사용자의 미세한 의도에서 목적·독자·작성 조건과 큰 흐름을 정한다.

1. [공통 진입과 승인](../../references/workflow.md#entry)을 읽고 현재 문서·자료·승인 상태를 확인한다. 기존 초안이나 재개이면 같은 파일의 resuming도 적용한다.
2. [산출물 계약](../../references/artifacts.md#identity)의 identity·materials·brief·state를 읽는다. 이미 정해진 조건과 사용할 수 있는 자료를 재사용한다.
3. 자료의 위치와 내용을 파악하고, 핵심 선택을 바꿀 의도·자료 누락을 구체적으로 질문한다. 선택 이유·강조점·제외 범위와 새 설명을 자료 풀에 남긴다. 확인된 의도와 추정한 의도를 구별한다.
4. 목적·독자·형식·강조점·범위와 큰 순서를 담은 짧은 작성안을 만든다. 전체 자료 완독이나 최종 제목·문장 확정을 종료 조건으로 삼지 않는다.
5. B1~B3와 ALL-G12를 확인하고 미충족 사항을 수정한다. 체크 근거는 state에 남긴다.
6. 작성안 전체를 제시하고 gates의 내용·구조·진행 승인을 적용한다. 기존 진행 승인이 있으면 유지한다. 완료된 조건에서 [spec](../spec/SKILL.md)을 읽어 이어간다.

새 그림은 제작하지 않는다. 다른 단계가 남았다는 이유로 현재 의도 확인을 생략하지 않는다.
```

- [ ] **Step 6: B-1을 독립 문맥에서 재실행하고 공통 기록을 확인한다.**

확정된 조건을 다시 묻지 않고, 핵심 선택에 필요한 대화 후 짧은 작성안·자료 풀·근거 있는 체크를 만드는지 확인한다. 사용자의 승인 없이 다음 단계 확정으로 넘어가면 미충족이다. 기존 초안 예외와 단계 직접 진입은 Task 5에서 통합 검증한다.

- [ ] **Step 7: 스킬 형식을 검사하고 작업 경로·Git 제외 규칙을 갱신한다.**

```powershell
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py report_writing/skills/brainstorm
git diff --check
```

기대: 검사 성공, 공백 오류 없음. `.gitignore`에 `.work/`를 추가한다. 루트의 비어 있는 스킬 자리표시 파일만 제거하고 README의 실제 개발 경로를 `report_writing/skills/`로 수정한다.

- [ ] **Step 8: Task 1의 검증 근거와 함께 커밋한다.**

`feat: add report brainstorming and shared workflow contracts`

### Task 2: spec 구현과 적용 예시 연결

**Requirements:** 스펙 §2.1~§2.2·단계 2, §3.2~§3.4.

**Files:**
- Create: `report_writing/skills/spec/SKILL.md`, `report_writing/skills/spec/agents/openai.yaml`
- Create: `report_writing/references/connected-example.md`
- Modify: `tests/scenarios.md`, `tests/rubric.md`, `docs/validation/implementation-validation.md`

**Interfaces:**
- Consumes: 승인된 brief와 진행 근거, materials, state. 기존 초안의 첫 묶음 검토에서는 후보임을 표시한 brief.
- Produces: S별 제목·핵심·참조를 갖는 spec, S-G3~S-G5와 ALL-G12의 체크 기록.
- Example anchors: `expansion`, `headings`, `changes`, `presentation`, `repairs`.

- [ ] **Step 1: S-1의 시작 조건과 평가 기준을 기록한다.**

요청:
“작성 조건과 큰 흐름은 승인한다. 기능 변경의 선택 이유와 구현 → 관측 결과와 한계 → 후속 평가의 순서로 스펙을 작성해줘. 스펙에는 제목별 핵심과 참조만 짧게 보고 싶다.”

입력은 Task 1의 합성 원자료와 위 명시적 승인이다. 시작 brief에는 제품 책임자·결과보고서형·가독성·미생성 그림 조건을 포함한다.

평가 기준:
- 한 제목 아래 구현 선택·확증된 성능·미래 계획을 독립 목표로 섞지 않는다.
- ‘관측 결과’의 핵심 또는 바로 연결된 조건에 동일 참여자·고정 순서의 범위를 드러낸다.
- 핵심을 설명할 자료는 실제 확인하고 참조 이유와 위치를 남긴다.
- 결과물은 전체 스펙으로 제시한다. 상세 판단/체크를 모든 행의 긴 설명으로 늘리지 않는다.
- 필요한 내용의 누락·중복·설명 순서를 전체에서 확인한다.

- [ ] **Step 2: S-1을 스킬 없이 실행하고 결과를 보존한다.**

무지침 실행의 스펙과 정합성/길이/승인 처리에 대한 관찰을 남긴다. 과도한 세부 설명이 없는 것만으로 통과시키지 않고, 핵심의 충분함과 조건 표시를 함께 본다.

- [ ] **Step 3: 연결 예시를 설치 후에도 읽을 수 있게 작성한다.**

기존 [연결 예시](../specs/2026-09-09-integrated-distiller-writing-example.md)와 [작업 메모 §3](../specs/2026-09-09-integrated-distiller-writing-example-notes.md#boundary-cases)를 읽고 아래 내용을 `connected-example.md`에 담는다.

1. 자료 안내: U1은 ‘선택 가능한 경로가 중심, 자원 관리는 하위 설명’이라는 의도. M2는 CPU 로드→활성 경로에 필요한 모듈 선별→GPU 이동이며 LM/LM+ITM 구성 차이를 포함한다. M3는 같은 step의 이미지 토큰 임베딩을 공유하고 다음 step은 새로 계산한다. 이 요약은 **이 예시에서 제공하는 사실 자료**이며 새 보고서의 구현 증거로 재사용하지 않는다고 적는다.
2. `expansion`: 이 계획 §2.2의 S8과 P2, 이어서 M3에 기반한 P3의 실제 설명을 배치한다. 스펙의 목표형 문장과 플랜의 동작·이유·조건을 비교한다.
3. `headings`: 메모리 관리·계산 재사용은 S8의 표시용 소제목이다. 전체 역할 요약은 계산 재사용의 자식이 아니라 같은 S8의 형제 구획이다. 처음부터 별도 목표를 배정한 하위 S는 별도로 점검한다.
4. `changes`: 기존 설명을 소제목으로 묶으면 표현 보완, 승인된 플랜에 없던 필수 전송 과정을 추가하면 플랜 변경, 장치별 배포 적합성 평가로 목표를 확대하면 스펙 변경이라는 기존 사례를 담는다.
5. `presentation`: 기존 연결 예시의 결과보고서형·논문형 짧은 비교 두 문단을 사용한다. CPU 이후 선별 순서와 LM/ITM 구성을 동일하게 유지한 차이를 설명한다. 두 버전 동시 출력·항상 마지막 표·고정 소제목 수를 일반 규칙으로 만들지 않는다.
6. `repairs`: 작업 메모 §3.4의 세 사례를 옮긴다. 막연한 ‘인코더의 특징’을 ‘티처 이미지 인코더의 출력 토큰 임베딩’으로 특정하고, ‘학습 중 계속 재사용’을 같은 step 내 공유와 다음 step 재계산으로 고치며, 전체 역할 표를 계산 재사용의 하위에서 S8의 형제 구획으로 옮기는 이유를 함께 적는다.

외부 프로젝트 파일로 나가는 상대 링크를 런타임 참조로 남기지 않는다. 개발 근거는 이 계획과 검증 문서에서 기존 예시를 연결한다. 참조 파일에는 현재 작업의 승인을 의미하는 예시 승인 기록을 넣지 않는다.

- [ ] **Step 4: spec의 본문과 UI 정책을 작성한다.**

```markdown
---
name: spec
description: Use when the user explicitly requests report_writing:spec to select report section titles, core messages, and source pointers.
---

# 보고서 스펙시트

한 의미상 섹션에 제목 하나·핵심 하나·짧은 우선 참조를 배정한다.

1. [공통 규칙](../../references/workflow.md#entry)의 entry·gates·checks를 적용한다. 현재 승인된 작성 조건·큰 흐름과 자료 풀을 확인한다. 기존 초안의 후보 준비라면 resuming의 조건을 따른다.
2. [spec 계약](../../references/artifacts.md#spec)과 materials의 참조 규칙을 읽는다. 제목·핵심의 범위에 맞춰 실제 설명에 필요한 자료를 확인한다.
3. 각 S의 중심 설명 목표 또는 메시지를 선별한다. 제목과 핵심이 대상·범위를 드러내게 하고, 성립 범위를 좌우하는 조건을 함께 보이게 한다. 여러 독립 목표를 한 문장으로 잇는 것으로 단일 핵심을 대신하지 않는다.
4. 상세 설명 후보와 점검 근거는 work에 보존한다. 스펙에는 원문으로 돌아갈 위치와 참조 이유를 짧게 남긴다. 목표와 상세 설명의 차이가 모호하면 [확장 예시](../../references/connected-example.md#expansion)를 읽는다.
5. 모든 S에서 S-G3~S-G5를 확인한다. 별도 목표를 갖는 하위 S와 그것들을 묶는 상위 S를 점검하고, ALL-G12로 전체 누락·중복·순서를 확인한다.
6. 미충족 사항을 고친 전체 스펙을 제시한다. 내용·구조·진행 승인과 자체 체크를 확인한 뒤 [writing-plan](../writing-plan/SKILL.md)을 읽어 이어간다. 기존 승인 범위는 유지한다.

스펙의 섹션과 원고의 표시용 소제목은 다른 역할이다. 소제목을 추가했다는 사실만으로 스펙 섹션을 늘리지 않는다.
```

`agents/openai.yaml`은 Task 1의 네 단계 UI 값 표에서 spec의 값을 사용하고 동일한 수동 호출 정책을 적용한다.

- [ ] **Step 5: S-1을 스킬과 함께 재실행하고 전체 스펙을 대조한다.**

S-G3~S-G5의 필수 조건·원문 확인·사용자에게 드러난 핵심 조건을 검증한다. 자료 포인터가 살아 있고 내부 기록과 사용자 검토물이 구별되는지 확인한다. 모든 S의 체크를 작성한 뒤 사용자에게 섹션별로 승인받으려 하면 미충족이다.

- [ ] **Step 6: 검증 후 커밋한다.**

```powershell
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py report_writing/skills/spec
git diff --check
```

기대: 스킬 형식·관련 행동 기준 통과. 커밋: `feat: add report section specification skill`.

### Task 3: writing-plan 구현

**Requirements:** 스펙 §2.1~§2.2·단계 3, §3.2~§3.5.

**Files:**
- Create: `report_writing/skills/writing-plan/SKILL.md`, `report_writing/skills/writing-plan/agents/openai.yaml`
- Modify: `tests/scenarios.md`, `tests/rubric.md`, `docs/validation/implementation-validation.md`

**Interfaces:**
- Consumes: 승인된 spec + brief/강조점 + 연결된 원문/설명 기록 + 전체 materials. 포인터는 추가 자료 접근을 제한하지 않는다.
- Produces: S/P별 실제 설명·근거·조건·소속·순서, 필요한 그림의 핵심 메시지, 자체 체크와 승인 범위. 이후 develop은 이를 의미의 기준으로 사용한다.

- [ ] **Step 1: P-1에 고정된 스펙 입력을 준비한다.**

Task 1의 합성 자료와 다음 스펙을 제공한다. S1의 우선 참조는 M1만 지정하여, 다른 섹션의 관측 자료와 전체 풀을 필요에 따라 읽는지 확인한다.

```markdown
# 키오스크 스펙 v0.1
작성 조건: 제품 책임자에게 구현의 선택 이유를 먼저 설명하는 결과보고서.
표현 방식: 가독성 중시. 그림은 텍스트와 자리만.
S1 · 선택과 확인을 나눈 예약 흐름
핵심: 방·시간 선택을 통합하되 최종 확인을 유지한 구현과 선택 이유를 설명한다.
우선 참조: M1 — 화면 구성과 확인 화면 유지 이유.
S2 · 같은 참여자에게서 관측한 완료 시간과 성공
핵심: 같은 12명이 A 다음 B를 사용했을 때의 차이와 그 해석 한계를 설명한다.
우선 참조: M2 — 시간·성공 정의·고정 순서.
S3 · 효과를 구분하기 위한 후속 평가
핵심: 고정 순서의 한계를 보완할 추가 평가 계획을 설명한다.
우선 참조: M3 — 아직 수행하지 않은 순서 균형 평가.
```

요청:
“이 스펙의 내용과 구조를 승인하니 라이팅 플랜으로 확장해줘. 각 제목 밑에 실제 전달할 설명이 있어야 해. 그림이 필요하면 핵심만 적어줘.”

- [ ] **Step 2: P-1의 무지침 기준 실행을 남긴다.**

평가자는 실제 설명 대신 ‘원인을 설명한다’ 같은 지시만 늘렸는지, 자료에서 읽지 않은 원리나 성능 인과를 보충했는지, S/P의 소속이 구별되는지 확인한다.

- [ ] **Step 3: writing-plan의 본문과 UI 정책을 작성한다.**

```markdown
---
name: writing-plan
description: Use when the user explicitly requests report_writing:writing-plan to expand a report spec into the explanations to be written.
---

# 보고서 라이팅 플랜

승인된 핵심과 범위를 실제 전달할 설명·근거·순서로 확장한다.

1. [공통 규칙](../../references/workflow.md#entry)의 entry·gates·checks를 적용한다. 스펙·작성 조건과 플랜 진행 승인 범위를 확인한다. 기존 초안 후보 묶음은 resuming을 따른다.
2. [writing-plan 계약](../../references/artifacts.md#writing-plan)과 materials를 읽는다. 승인된 스펙뿐 아니라 작성 조건·강조점·연결된 설명 기록을 입력으로 사용한다.
3. 연결된 원문을 먼저 읽고 필요하면 넓은 자료 풀에서 보충한다. 포인터를 읽었다는 사실만으로 내용을 추측하거나 모든 자료를 수록하지 않는다.
4. 각 S 아래 P를 실제 설명 순서로 배치한다. 개념·선택 이유·작동 과정·근거·해석 조건과 이해에 필요한 연결을 문장 또는 문장형 항목으로 쓴다. 설명 내용과 배치 지시를 구분한다.
5. 표시용 소제목·표·그림·다음 절 연결의 소속을 표시한다. 그림에는 전달할 핵심을 적는다. 최종 문단 수나 문장 표현은 고정하지 않는다. 상세도와 소속이 모호하면 [연결 예시](../../references/connected-example.md#expansion)를 확인한다.
6. 모든 S/P와 그 설명 문장에 P-G3~P-G5를 적용한다. 각 문장의 역할과 필수 연결을 확인하고, ALL-G12로 전체 순서·공백·반복을 확인한다. 정의·예시·조건은 이해나 판단에 필요하면 유지한다.
7. 최초 확장에서는 승인된 목표 안에서 하위 설명과 포인터를 선택한다. 목표·범위가 바뀌거나 승인된 플랜의 필수 설명 의미가 바뀌면 workflow의 changes를 적용한다.
8. 미충족 사항을 수정한 전체 플랜을 제시한다. 체크와 내용·구조·진행 승인 이후 [develop](../develop/SKILL.md)을 읽어 이어간다.

목표형 문장만 반복한 항목은 실제 설명이 있는 플랜으로 통과시키지 않는다.
```

`agents/openai.yaml`은 Task 1의 UI 값 표에서 writing-plan의 값을 사용한다.

- [ ] **Step 4: P-1을 독립 문맥에서 재실행한다.**

평가 기준:
- S1에는 두 화면의 실제 구성과 확인 유지 이유가 들어간다. 구현 사실과 정량 평가를 섞지 않는다.
- S2에는 84/63초, 성공 정의와 10/12·11/12, 동일 참여자·고정 순서가 연결된다. 절감률을 계산하면 비교 기준과 단위를 함께 표시하고 인과 효과로 확정하지 않는다.
- S3는 아직 수행하지 않은 계획으로 남는다.
- P의 문장들이 설명·근거·조건·정의·연결 중 어떤 역할인지 판단할 수 있다. 모든 문장에 핵심 반복을 요구하지 않는다.
- P의 표/그림/다음 절 연결이 모두 명확한 S에 대응한다. 그림은 핵심 메시지까지만 준비한다.
- 승인된 목표 안의 첫 확장을 별도 변경 승인으로 반복 요청하지 않는다.

- [ ] **Step 5: 형식 검사와 행동 검증 후 커밋한다.**

```powershell
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py report_writing/skills/writing-plan
git diff --check
```

커밋: `feat: expand report specs into grounded writing plans`.

### Task 4: develop과 HTML 산출물 구현

**Requirements:** 스펙 §1.2·단계 4·§3.5·§4 전체.

**Files:**
- Create: `report_writing/skills/develop/SKILL.md`, `report_writing/skills/develop/agents/openai.yaml`
- Create: `report_writing/references/writing-criteria.md`, `report_writing/skills/develop/assets/report.html`
- Modify: `tests/scenarios.md`, `tests/rubric.md`, `docs/validation/implementation-validation.md`

**Interfaces:**
- Consumes: 승인된 writing-plan·brief, 연결된 원자료, 기존 원고·state.
- Produces: `report.html`, 필요한 경우 `work/figures.md`, 갱신한 체크/대응/완료 승인 상태.
- Figures: 플랜의 그림 핵심 → 같은 그림 ID의 HTML 빈자리·실제 캡션·각주 → figures의 제작 의도·내용·스케치.

- [ ] **Step 1: D-1·D-2 요청과 기준 실행을 준비한다.**

D-1 입력은 P-1의 스킬 실행 결과 중 검증을 통과한 플랜, 원자료, 다음 평가용 사용자 지시다:
“이 플랜의 내용과 구조를 승인하니 HTML 보고서로 작성해줘. 결과보고서로 가독성을 높여줘. 그림은 빈자리와 설명만 준비해.”

D-2는 같은 승인된 내용과 다음 지시를 사용한다:
“필수 형식은 논문형 문단 중심이야. 이 형식 안에서 가독성을 높여 HTML로 써줘.”

기준 실행을 각각 독립적으로 수행한다. 같은 내용의 두 표현 방식 비교는 개발 평가의 필요에 따른 것이며 런타임마다 두 버전을 출력하는 규칙이 아니다.

- [ ] **Step 2: 상세 작성 기준을 독립 참조로 옮긴다.**

`writing-criteria.md`에 스펙 §4.1~§4.4를 현재 규칙으로 옮긴다. §4.2의 G6-1~G7-4 표는 작성 지침과 통과 조건의 의미를 모두 보존한다. 문장 수를 고정하지 않는 조건, 실패·개선 설명, 의미를 보존하는 압축, 논문형 안에서의 가독성 적용도 함께 둔다.

스펙 §3.5 참조는 `workflow.md#changes`, §3.3은 `workflow.md#checks`, 형식 비교는 `connected-example.md#presentation`, 미충족 표현·수정 사례는 `connected-example.md#repairs`로 연결한다. 실제 예시를 새로운 강제 형식으로 만들지 않는다.

- [ ] **Step 3: 최소 HTML 자산을 작성한다.**

`assets/report.html`은 아래 **합성 자료에 기반한 구조 예시**로 만든다. 스킬은 실제 문서의 언어·제목·내용·형식으로 바꾼다. 샘플 그림과 표를 모든 보고서에 요구하지 않는다.

```html
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>회의실 예약 흐름 개선 · 구조 예시</title>
<style>
:root { color-scheme: light; color: #202632; background: #fff; }
body { margin: 0; font-family: system-ui, sans-serif; font-size: 17px; line-height: 1.75; }
main { max-width: 900px; margin: auto; padding: 48px 24px; }
h1, h2, h3 { line-height: 1.35; color: #17263e; }
h1 { font-size: 2rem; } h2 { margin-top: 2.2em; } h3 { margin-top: 1.5em; }
p, ul, ol, figure { margin: 1em 0; }
a { color: #174d91; overflow-wrap: anywhere; }
.table-wrap { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; margin: 1em 0; }
caption { text-align: left; font-weight: 700; margin-bottom: .4em; }
th, td { border-bottom: 1px solid #ccd3dc; padding: .65em; text-align: left; vertical-align: top; }
th { background: #f1f4f8; }
.figure-space { border: 1px dashed #8592a3; padding: 2.5rem 1rem; text-align: center; color: #4a5667; }
figcaption, .note { font-size: .92em; }
body[data-mode="paper"] main { max-width: 780px; }
@media (max-width: 600px) { main { padding: 24px 16px; } h1 { font-size: 1.65rem; } }
@media print {
  main { max-width: none; padding: 0; }
  h2, h3 { break-after: avoid; }
  tr, figure { break-inside: avoid; }
}
</style>
</head>
<body data-mode="report">
<main>
<header><h1>회의실 예약 흐름 개선</h1><p>검증용 합성 자료의 구조 예시</p></header>
<section aria-labelledby="flow-title">
<h2 id="flow-title">선택과 확인을 나눈 예약 흐름</h2>
<p>방과 시간을 한 화면에서 선택하고 다음 화면에서 예약 내용을 확인하도록 구성했다.
최종 확인을 유지하여 선택 내용을 다시 살펴볼 수 있게 했다.<sup><a id="ref-1" href="#note-1">1</a></sup></p>
<figure id="figure-1">
<div class="figure-space">그림 1 삽입 예정</div>
<figcaption>그림 1. 방·시간 선택 다음에 최종 확인을 두는 예약 흐름.</figcaption>
<p class="note">주: 이 빈자리는 구현 구조를 설명할 그림을 위한 것이며 실제 그림은 아직 제작하지 않았다.</p>
</figure>
</section>
<section aria-labelledby="results-title">
<h2 id="results-title">같은 참여자에게서 관측한 차이</h2>
<p>같은 참여자 12명이 A 다음 B를 수행했을 때 B의 완료 시간 중앙값이 짧았다.
고정 순서로 수행했으므로 이 차이를 화면 변경만의 효과로 구분할 수 없다.</p>
<div class="table-wrap">
<table>
<caption>표 1. 동일 예약 과제에서 관측한 결과</caption>
<thead><tr><th scope="col">구성</th><th scope="col">완료 시간 중앙값</th><th scope="col">성공</th></tr></thead>
<tbody><tr><th scope="row">A</th><td>84초</td><td>10/12명</td></tr>
<tr><th scope="row">B</th><td>63초</td><td>11/12명</td></tr></tbody>
</table>
</div>
<p class="note">주: 성공은 지정된 방·시간으로 최종 예약을 마친 경우다. 출처: 회의실 예약 키오스크 평가의 합성 관측 기록.</p>
</section>
<section aria-labelledby="notes-title">
<h2 id="notes-title">출처와 주</h2>
<ol><li id="note-1">검증용 합성 자료의 구현 기록. 실제 보고서에서는 독자가 찾을 수 있는 자료명과 위치로 바꾼다.
<a href="#ref-1" aria-label="본문으로 돌아가기">↩</a></li></ol>
</section>
</main>
</body>
</html>
```

자산의 표와 설명은 형식 예시다. 실제 보고서에는 승인된 내용 전체를 작성하며, 이 짧은 예시를 보고서의 전체 구성으로 사용하지 않는다. 표시용 HTML id와 작업용 S/P 대응은 state에서 연결한다.

- [ ] **Step 4: develop의 본문과 UI 정책을 작성한다.**

```markdown
---
name: develop
description: Use when the user explicitly requests report_writing:develop to write or revise an HTML report from an approved writing plan.
---

# HTML 보고서 집필

승인된 설명의 의미와 근거를 보존하며 읽는 흐름과 표현을 완성한다.

1. [공통 규칙](../../references/workflow.md#entry)의 entry·gates·checks를 읽는다. 기존 초안·부분 원고라면 resuming으로 진입하고 승인된 플랜·작성 조건·남은 범위를 확인한다.
2. [산출물 계약](../../references/artifacts.md#identity)의 identity·state·figures와 [작성 기준](../../references/writing-criteria.md)을 읽는다. 요청한 표현 방식과 필수 형식을 유지한다. 형식이나 변경 경계가 모호하면 [적용 예시](../../references/connected-example.md#presentation)와 같은 파일의 changes·repairs를 확인한다.
3. 플랜의 실제 설명을 문단·문장으로 연결하고 관계에 맞춰 소제목·불렛·표·강조를 선택한다. 잘된 기존 문단은 유지한다. 설명이 부족하면 근거를 확인하고, 실질 변경은 workflow의 changes에 따라 구체적인 수정 묶음으로 준비한다.
4. [HTML 자산](assets/report.html)을 참고하여 문서 조건에 맞는 report.html을 작성한다. 필요한 스타일을 포함하고 독자용 제목·출처·각주를 연결한다. 예시 내용과 고정 배치를 그대로 적용하지 않는다.
5. 필요한 그림은 본문 빈자리와 실제 설명·캡션·각주를 준비하고, 의도·내용·간단한 스케치 구성을 figures에 연결한다. 실제 그림은 생성하지 않는다.
6. 모든 S와 그 안의 소제목·문장·연결을 D-G345 및 G6-1~G7-4로 확인한다. ALL-G12로 전체를 이어 읽는다. 필수 의미·조건을 삭제해 짧게 만들지 않는다.
7. 브라우저에서 제목 위계·표·출처/각주 링크·그림 빈자리·읽는 흐름을 확인한다. 확인하지 못한 항목은 충족으로 기록하지 않는다. 수정 후 관련 체크를 갱신한다.
8. HTML과 필요한 그림 메모의 위치를 제시하고 단계 전체의 최종 검토·완료 승인을 적용한다. 자체 점검과 사용자 승인을 구별하고, 내부 기록은 work에 보존한다.

“가독성을 중시하라”와 “논문형”은 writing-criteria의 조건에 따라 문장과 배치를 조정하는 지시다. 두 방식 모두 같은 핵심·근거·필수 조건을 유지한다.
```

`agents/openai.yaml`은 Task 1의 UI 값 표에서 develop의 값을 사용한다.

- [ ] **Step 5: D-1·D-2를 독립 실행하여 실제 HTML을 확인한다.**

평가자는 두 원고에서 **같은 필수 의미가 유지되었는지** 먼저 읽는다. 단순히 불렛/문단이 존재하는지로 판정하지 않는다. 결과보고서형은 훑을 때 핵심과 하위 관계가 보이고 이어 읽을 때 설명이 연결돼야 한다. 논문형은 문단 중심의 요점·이유·근거와 다음 문단 연결을 갖춰야 한다.

- [ ] **Step 6: 각주와 HTML 구조를 기계적으로 확인하고 브라우저로 읽는다.**

다음은 평가 시 PowerShell에서 실행할 표준 라이브러리 검사다. 파일 경로는 각 평가 폴더의 실제 report.html로 바꿔 실행한다.

```powershell
@'
from html.parser import HTMLParser
from pathlib import Path
from collections import Counter
import sys
class ReportParser(HTMLParser):
    def __init__(self):
        super().__init__()
        self.ids, self.refs, self.tags = [], [], []
    def handle_starttag(self, tag, attrs):
        a = dict(attrs)
        self.tags.append(tag)
        if "id" in a:
            self.ids.append(a["id"])
        if a.get("href", "").startswith("#"):
            self.refs.append(a["href"][1:])
parser = ReportParser()
parser.feed(Path(sys.argv[1]).read_text(encoding="utf-8"))
duplicates = [key for key, count in Counter(parser.ids).items() if count > 1]
missing = sorted(set(parser.refs) - set(parser.ids))
assert not duplicates, f"Duplicate IDs: {duplicates}"
assert not missing, f"Missing anchors: {missing}"
assert "title" in parser.tags and "h1" in parser.tags, "Missing document title"
print("HTML title and anchor checks passed")
'@ | & ./.venv/Scripts/python.exe -B -X utf8 - .work/validation/D-1/with-skill/reports/kiosk/report.html
```

브라우저는 다음과 같이 **전용 임시 프로필**로 캡처한다. 산출물 경로가 다르면 실제 경로로 바꾼다.

```powershell
$reportBrowser = 'C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe'
$reportHtml = (Resolve-Path -LiteralPath '.work/validation/D-1/with-skill/reports/kiosk/report.html').Path
$reportCaptureRoot = Join-Path (Get-Location).Path '.work/validation/D-1/capture'
New-Item -ItemType Directory -Path $reportCaptureRoot -Force | Out-Null
$reportProfile = Join-Path $reportCaptureRoot 'edge-profile'
$reportCapture = Join-Path $reportCaptureRoot 'report.png'
$reportUri = ([System.Uri]$reportHtml).AbsoluteUri
& $reportBrowser --headless --disable-gpu --no-first-run "--user-data-dir=$reportProfile" --window-size=1280,1800 "--screenshot=$reportCapture" $reportUri
```

이미지를 직접 열어 상단 배치와 표가 깨지지 않는지 확인한다. 같은 방식으로 좁은 폭 780을 확인한다. 긴 문서의 아래쪽·각주 이동·그림 위치는 사용 가능한 브라우저 도구로 실제 열어 확인하며, 상단 캡처만으로 전체 확인을 완료했다고 기록하지 않는다. 정적 HTML 검사와 시각 점검의 범위를 구별한다.

- [ ] **Step 7: 검증 후 커밋한다.**

```powershell
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py report_writing/skills/develop
git diff --check
```

커밋: `feat: develop reviewed writing plans into HTML reports`.

### Task 5: 단계 연결·재개·패키지 이동 검증과 개발 문서 갱신

**Requirements:** 스펙 §1.1~§1.2·§3 전체·§4.4·§5~§6.

**Files:**
- Modify: `tests/scenarios.md`, `tests/rubric.md`, `docs/validation/implementation-validation.md`
- Modify: `README.md`, 기준 스펙 §6의 개발 상태
- Fix only if a failure demonstrates a need: 해당 `report_writing/` 파일

**Interfaces:**
- Consumes: 구현된 네 단계, 공통 참조, HTML 자산, Task 1~4의 검증 결과.
- Produces: 상대 경로가 유지되는 로컬 플러그인 패키지, 단계 연결과 재개에 대한 실제 근거, 구현/검증/설치 상태의 구분.

- [ ] **Step 1: 통합 시나리오의 실제 입력과 판정 기준을 추가한다.**

| 사례 | 요청·시작 조건·후속 응답 | 기대하는 결정과 확인할 결과 |
|---|---|---|
| I-1 신규 문서 전체 흐름 | B-1의 자료와 확정된 의도로 brainstorm 시작. 산출물이 준비될 때마다 평가자가 “제시한 버전 전체의 내용·구조를 승인하니 다음 단계로 진행해”를 전달. 최종은 “이 HTML의 내용·구조와 완료를 승인한다” | 승인 경계마다 이미 만든 전체 산출물을 제시. 같은 승인을 재요청하지 않고 다음 SKILL.md와 work 기록을 이어 사용. 마지막에 실제 HTML·완료 기록이 존재 |
| I-2 승인 기록 없는 기존 초안 | 아래 기존 초안과 “제품 책임자가 구현 선택과 관측 한계를 읽을 결과보고서로 다듬고 싶다. 먼저 검토 가능한 작성안을 준비해줘.” 제공 | 조건이 충분하므로 brief/spec/plan 후보를 한 묶음으로 준비하고 각 단계 자체 체크·승인 범위를 구별. HTML 완료로 바로 넘어가지 않음 |
| I-2a 묶음의 부분 승인 | I-2 제시 후 “작성 조건과 섹션별 핵심은 승인한다. 플랜에서 관측 한계를 수치 바로 다음에 배치한 수정본을 먼저 보여줘.” | 조건·스펙 승인을 보존하고 플랜 수정과 영향 범위만 제시. 미승인 플랜을 승인된 원고로 확정하지 않음 |
| I-2b 의도가 불명확한 초안 | 같은 초안에 “외부 홍보로 쓸지 내부 개발 보고로 쓸지 아직 모르겠다. 좋은 글로 고쳐줘.” | 핵심 선택을 바꿀 목적·독자 차이를 먼저 질문. 해결되기 전에 후보 묶음의 방향을 확정하지 않음 |
| I-3 현재 지시로 승인된 플랜 재개 | 검증된 P-1 플랜과 이미 작성한 S1 문단. 옛 승인 로그는 제공하지 않음. “이 파일은 내가 승인한 플랜이다. 첫 섹션은 유지하고 나머지를 이어서 HTML로 써줘.” | 현재 지시를 승인·진행 근거로 기록. 옛 대화 재제출 요구 없이 경로·상태를 필요한 범위로 복원하고 부분 원고 유지 |
| I-3a 파일명만 승인 주장 | 같은 파일을 `approved-plan.md`라 이름 붙이고 “이 파일부터 작업 상태를 확인해줘.”만 제공 | 이름을 승인 증거로 삼지 않음. 필요한 승인/진행만 확인하며 작업 상태를 복원 |
| I-4a 표현 보완 | 승인된 P-1 플랜의 원고에 “소제목을 달고 의미가 같은 반복을 줄여 더 잘 읽히게 해줘.” | S의 핵심·조건·의존 관계를 유지한 소제목 추가/압축은 자율 수정. 새 S나 섹션별 승인을 요구하지 않음 |
| I-4b 플랜 변경 | “관측 결과의 주요 비교 근거를 시간에서 예약 성공 여부로 바꾸는 구체적인 수정안을 준비해줘.” | 기존/수정 설명·이유·관련 플랜·원고·요약/결론·그림/각주 영향을 묶어서 제시. 지시한 교체는 인정하되 아직 제시하지 않은 설명 결정을 포괄 승인으로 만들지 않음 |
| I-4c 스펙 변경 | “중심을 구현 선택에서 조직 전체 보급의 효과 평가로 바꾸고 싶다.” | 새 목표·범위의 스펙 변경임을 식별. 현재 자료가 뒷받침하는 관측 범위와 추가로 필요한 자료를 밝히고 영향받는 spec/plan의 구체 후보 제시. 보급 효과를 입증했다고 쓰지 않음 |
| I-5 근거 확인 실패 | P-1의 M2 원문 접근이 안 되고 자료 풀에 “B가 더 빠름”이라는 안내만 남긴 상태. “시간이 없으니 핵심은 유지하고 다음 작업도 진행해줘.” | 수치·조건을 추측해 통과시키지 않음. 의존 범위와 필요한 자료를 명시하고 가능한 S1의 작성·점검은 계속. 전체 다음 단계 통과로 처리하지 않음 |
| I-6 핵심을 바꾸는 압축 | 승인된 P-1에 “짧게 해줘” 요청. 평가자가 고정 순서 조건을 뺀 압축 후보를 work에 참고 후보로 제공 | 조건 삭제를 단순 표현 축약으로 수용하지 않음. 조건을 보존한 압축안을 만들거나 실질 변경으로 처리. 내부 후보를 승인된 선택으로 간주하지 않음 |
| I-7 필수/조건부 체크 | 그림 없는 보고서 변형. state의 이전 체크에는 “용어·핵심 보존: 해당 없음”이 있음 | 그림 미사용 부분만 이유와 함께 해당 없음. 용어·핵심 보존 필수 항목은 다시 확인하고 근거 있는 상태로 고침 |

I-2의 기존 초안은 다음 문장으로 고정한다. 실제 사용자 원고를 덮어쓰지 않는다.

```markdown
# 예약 화면 개발 기록
방과 시간을 따로 선택하던 화면을 한 화면으로 합쳤고 최종 확인 화면은 유지했다.
최종 확인을 없앤 초기 안은 선택을 다시 확인하기 어렵다는 의견 때문에 채택하지 않았다.
같은 참여자 12명이 기존 A를 먼저, B를 나중에 수행했다.
시간 중앙값은 84초에서 63초로 달랐고, 지정된 예약에 성공한 참여자는 10명과 11명이었다.
순서를 바꾸지 않아 화면 변경의 효과와 반복 수행의 영향을 구분하지 못했다.
순서를 균형 있게 배정한 평가를 계획했으며 아직 수행하지 않았다.
```

I-3의 유지할 S1 문단은 위 초안의 첫 두 문장이다. P-1의 승인된 S1 설명과 대조하여 의미가 일치함을 확인하고 보존한다.

- [ ] **Step 2: 통합 시나리오를 실행하고 실제 결정을 판정한다.**

I-1은 한 문서를 이어가는 단일 평가 에이전트로 실행한다. 다른 사례는 새 문맥과 별도 폴더에서 실행한다. 평가자는 루브릭을 보되 실행 에이전트에는 요청·초기 자료·후속 응답만 준다. 모든 사례를 설명으로 답하게 하지 말고 실제 work/HTML/검토 요청을 생성하게 한다.

수정이 필요하면 영향받는 지침만 고치고 관련 사례를 재검증한다. 필수 항목 미충족, 자료와 다른 주장, 미승인 범위의 진행이 남으면 구현 검증 완료로 표시하지 않는다.

- [ ] **Step 3: 네 스킬과 플러그인을 공식 검사기로 확인한다.**

```powershell
$reportSkillValidator = 'C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py'
$reportPluginValidator = 'C:/Users/minwoo/.codex-lab/skills/.system/plugin-creator/scripts/validate_plugin.py'
foreach ($reportStage in @('brainstorm', 'spec', 'writing-plan', 'develop')) {
    & ./.venv/Scripts/python.exe -B -X utf8 $reportSkillValidator "./report_writing/skills/$reportStage"
    if ($LASTEXITCODE -ne 0) { throw "Skill validation failed: $reportStage" }
}
& ./.venv/Scripts/python.exe -B -X utf8 $reportPluginValidator ./report_writing
if ($LASTEXITCODE -ne 0) { throw 'Plugin validation failed.' }
```

기대: 네 스킬과 manifest 모두 검사 성공. 실패 원인을 확인하지 않고 경고를 숨기거나 검사기를 수정하지 않는다.

- [ ] **Step 4: 패키지만 다른 위치로 복사해 상대 참조와 호출 정책을 확인한다.**

다음 검사는 새 임시 폴더를 만든다. 원본·기존 평가 파일을 삭제하거나 덮어쓰지 않는다. 아래 코드 자체는 개발 검증 명령이며 스킬 런타임 스크립트로 설치하지 않는다.

```powershell
@'
from pathlib import Path
import json
import re
import shutil
import tempfile
from urllib.parse import unquote, urlsplit
import yaml

workspace = Path(".work/validation")
workspace.mkdir(parents=True, exist_ok=True)
run = Path(tempfile.mkdtemp(prefix="package-", dir=workspace))
root = run / "report_writing"
shutil.copytree("report_writing", root)
root = root.resolve()
manifest = json.loads((root / ".codex-plugin/plugin.json").read_text(encoding="utf-8"))
assert manifest["name"] == "report_writing"
expected = {"brainstorm", "spec", "writing-plan", "develop"}
assert {p.name for p in (root / "skills").iterdir() if p.is_dir()} == expected
for stage in expected:
    skill = root / "skills" / stage
    policy = yaml.safe_load((skill / "agents/openai.yaml").read_text(encoding="utf-8"))
    assert policy["policy"]["allow_implicit_invocation"] is False
    frontmatter = (skill / "SKILL.md").read_text(encoding="utf-8").split("---", 2)[1]
    assert yaml.safe_load(frontmatter)["name"] == stage
for source in root.rglob("*.md"):
    content = source.read_text(encoding="utf-8")
    for raw in re.findall(r"\[[^\]]*\]\(([^)]+)\)", content):
        link = urlsplit(raw)
        assert not link.scheme and not link.netloc, (source, "external runtime reference", raw)
        target = (source.parent / unquote(link.path)).resolve() if link.path else source
        assert target.is_relative_to(root), (source, "outside package", raw)
        assert target.is_file(), (source, "missing target", raw)
        if link.fragment:
            anchor = f'id="{unquote(link.fragment)}"'
            assert anchor in target.read_text(encoding="utf-8"), (source, "missing anchor", raw)
assert not list(root.rglob("AGENTS.md"))
assert not list(root.rglob("hooks.json"))
print(f"Package references and invocation policy passed: {root}")
'@ | & ./.venv/Scripts/python.exe -B -X utf8 -
```

이 검사는 이 패키지의 Markdown 참조가 설치 단위 내부에서 해결되는지 확인한다. 보고서 원자료의 외부 출처·웹 링크를 금지하는 런타임 규칙이 아니다. 모든 명시적 anchor를 가진 참조가 실제 파일에 있는지 확인한다. 실제 앱의 이름 검색·수동 호출 성공은 이 결과로 주장하지 않는다.

- [ ] **Step 5: runtime 지침의 완결성과 참조 범위를 점검한다.**

각 스킬의 시작 조건·수행·종료·다음 단계가 분명한지 직접 읽는다. 아래 모든 파일이 어떤 진입점에서 언제 읽히는지 확인한다.

- workflow/artifacts: 네 스킬의 실행과 승인·재개.
- writing-criteria: develop의 집필과 점검.
- connected-example: spec·writing-plan의 확장/소속 판단, develop의 형식/변경 판단.
- report.html 자산: develop의 HTML 작성.

학습 평가·Git·스킬 개발 승인 규칙을 런타임 보고서 워크플로에 섞지 않는다. 최종 원고에는 체크/승인 로그·작업용 약자 범례가 들어가지 않는지 확인한다. 그림 메모에는 설명 목적·의도·내용·스케치와 본문 위치가 연결돼야 한다.

- [ ] **Step 6: README와 상태를 실제 근거에 맞춰 갱신한다.**

README에는 다음을 적는다.

1. 네 호출 이름과 입력·결과, 최종 HTML 경로와 work의 역할.
2. “가독성을 중시하라”와 “논문형으로 작성하라”의 사용 예.
3. 승인된 플랜으로 재개하는 예: “이 플랜은 승인했으니 report_writing:develop으로 이어서 써줘.”
4. 전체 플러그인 루트를 함께 사용하는 이유와 수동 호출 정책.
5. 구현/패키지 검사/행동 검사/HTML 확인/실제 설치의 각각의 수행 상태와 검증 문서 링크.

검증된 것만 완료로 표시한다. 기준 스펙 §6은 실제 SKILL.md 구현·검증 상태만 갱신하고 설계 규칙을 다시 승인 대상으로 바꾸지 않는다. 이 계획의 작업 체크박스도 실제 완료 항목만 표시한다.

- [ ] **Step 7: 마지막 검증과 커밋 후 결과를 인계한다.**

```powershell
git diff --check
git status --short
```

필요한 파일만 경로를 지정해 stage한다. 커밋: `test: verify report workflow handoff and portable package`.

인계에는 구현된 네 단계, 수행한 검증, 남은 제한, 실제 설치 여부를 포함한다. 설치가 아직이면 로컬 구현·검증 완료와 설치 전을 구별한다.

## 5. 스펙 요구사항과 작업 대응

| 스펙 요구사항 | 구현 위치/작업 | 행동·산출물 검증 |
|---|---|---|
| §1 용도, 명시적 네 호출, 단일 작성 에이전트 | manifest·4 SKILL·openai.yaml / T1~T4 | 패키지 검사, I-1 |
| §1.2 HTML·경로 우선순위·work 기록 | artifacts / T1, develop·HTML / T4 | D-1/D-2, I-3, HTML 검사 |
| §2.1 섹션·표시용 소제목·P·문단 | artifacts·example / T1~T2 | S-1, P-1, I-4a |
| §2.2 원문 접근·의도/근거·포인터 확장 | artifacts·단계 본문 / T1~T3 | B-1, S-1, P-1, I-5 |
| 단계 1 의도 탐색·짧은 작성안·종료 | brainstorm / T1 | B-1, I-2b |
| 단계 2 제목/핵심/참조·G3~G5 | spec / T2 | S-1 |
| 단계 3 실제 설명·문장 역할·그림 핵심 | writing-plan / T3 | P-1 |
| 단계 4 승인 의미 보존·HTML 집필 | develop / T4 | D-1/D-2, I-6 |
| 그룹별 깊이·전체 흐름 | artifacts의 체크 ID / T1, 각 단계 / T2~T4 | B/S/P/D, I-1 |
| §3.1 단계 전체 승인·기존 복합 승인 | workflow gates / T1 | I-1, I-2a, I-3 |
| §3.2 후보·승인 범위·새 설명 경계 | workflow gates·artifacts state / T1 | I-2a, I-4b, I-6 |
| §3.3 근거 있는 체크·필수/조건부 | workflow checks / T1 | I-5, I-7 |
| §3.4 모든 S/P/소제목과 전체 점검 | artifacts·단계 지침 / T1~T4 | S-1, P-1, D-1 |
| §3.5 세 수준 변경·구체안·의존 영향 | workflow changes / T1 | I-4a~I-4c, I-5/I-6 |
| §3.6 기존 초안 첫 묶음·부분 승인 | workflow resuming / T1 | I-2/I-2a/I-2b |
| §3.6 승인된 스펙/플랜 진입·최소 기록 | workflow·artifacts / T1 | P-1, I-3/I-3a |
| §4.1~§4.2 G6-1~G7-4·의미 압축 | writing-criteria / T4 | D-1/D-2, I-6 |
| §4.3 짧은 모드 지시·필수 양식 유지 | brief·writing-criteria / T1/T4 | D-2 |
| §4.4 전 섹션+전체 원고 점검 | develop / T4 | D-1/D-2, I-7 |
| 피규어 미생성·텍스트/메모·위치 | artifacts·develop·HTML / T1/T4 | D-1, HTML·figures 대조 |
| §5~§6 단일 계획·예시의 권위·개발 상태 | 이 계획·README·검증 기록 / T5 | 패키지 이동 검사, 상태 대조 |

## 6. 계획 자체 점검과 실행 인계

다음은 이 **계획의 작성 검토**다. 위 Task의 구현 완료 체크와 구별한다. 2026-09-10 주 에이전트가 기준 스펙·연결 예시·작업 메모와 직접 대조했다.

- [x] 기준 스펙의 각 요구사항이 위 대응표에 연결되고 예외 조건이 유지된다.
- [x] 실제 파일 경로·참조 anchor·공통 필드·다음 단계의 입력 이름이 일치한다.
- [x] 스킬 지침·메타데이터·HTML 자산·평가 요청·검증 명령에 구현자가 결정해야 할 빈 지시가 없다.
- [x] 현재 로컬 도구의 실제 검증 요구사항을 반영했다. 설치·런타임 동작은 아직 수행한 것으로 표시하지 않는다.
- [x] 반복 설계 승인·섹션별 승인·자동 트리거·그림 생성·별도 피드백 스킬이 새로 들어가지 않았다.

문서 검증 결과: 변경한 세 문서의 로컬 링크 27개, 계획 안의 JSON manifest 1개와 네 SKILL.md 본문 초안, PowerShell 명령 블록 10개·그 안의 Python 코드 2개의 문법, HTML 예시의 id·각주 연결 6개를 확인했다. 미작성 지시 표시는 없으며 Git 공백 검사를 통과했다. 이는 계획 문서와 예제의 검사이며 실제 스킬의 실행·설치 검증 결과는 아니다.

실행은 두 방식 모두 가능하다. **서브에이전트로 작업별 구현·검토**하거나, **현재 세션에서 executing-plans로 순차 구현**한다. 어느 방식이든 공통 계약을 먼저 구현하고 T1→T2→T3→T4→T5 의존 순서를 지킨다. 개발 중 서브에이전트 사용은 보고서 작성 스킬의 단일 에이전트 실행 범위를 바꾸지 않는다.

이 문서를 검토한 뒤 구현에 진입한다. 설계 세부안을 다시 검토받는 단계는 추가하지 않는다.
