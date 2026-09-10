# 구현 검증 기록

## 공통 조건

- 날짜: 2026-09-10
- 평가 산출물: `.work/validation/<case>/<run>/` (Git 제외)
- 행동 평가: `tests/scenarios.md`와 `tests/rubric.md`에 따라 응답 원문과 생성 파일을 의미 수준에서 확인
- 형식 평가: Python 3.12, PyYAML 6.0.3, 공식 검사기를 `-B -X utf8`로 실행

## B-1 · brainstorm

### 스킬 없는 기준 실행

- 모델·추론: `gpt-6-astra`, high, 새 문맥
- 제공 조건: B-1 요청과 `tests/fixtures/kiosk/materials.md`의 원자료만 제공. 대상 스킬·구현 계획·설계 스펙·루브릭·정답은 미제공
- 원문: `.work/validation/B-1/control/response.md`, `.work/validation/B-1/control/response-followup.md`
- 생성물: `.work/validation/B-1/control/brief.md`

최초 응답은 기능 변경 중심의 선호안과 성능 관측 중심의 대안을 함께 제시했으나, 사용자가 미정이라고 밝힌 중심 강조점을 질문으로 확인하지 않았다. 후속 응답 뒤에는 방·시간 통합 구현과 최종 확인 유지 이유를 중심에 두고, 관측 결과를 그 뒤에 배치했다. A→B 고정 순서의 한계와 추가 평가가 아직 계획이라는 상태를 정확히 보존했고 인과 주장을 만들지 않았다. M4는 본문에서 제외하면서 관계가 확인되지 않았다는 이유를 남겼다.

내용 품질과 과정 기록을 구분해 판정했다. 의도·관측·계획의 구분, 원자료와 한계 보존, 읽을 수 있는 큰 흐름은 충족했다. 최초 불확실성을 질문으로 해소하지 않은 행동은 B1/B3의 미충족이다. 생성된 brief와 응답에는 문서 버전, 선행 버전, 승인 범위, B1~B3·ALL-G12의 상태와 근거를 보존하는 materials/state 기록이 없었다. 이는 글 품질의 실패가 아니라 지속 가능한 인계 계약의 차이다.

원자료에는 방·시간 통합 자체의 동기가 없다. 스킬 실행이 이를 발견해 확인했으므로, 비교의 입력 대칭성을 유지하기 위해 “원자료에 없는 동기를 추가할 필요는 없고, 확인된 화면 구성 변화와 최종 확인 유지 이유 중심; 지금은 작성안 검토만, 스펙 진행 승인 아님”이라는 보완 지시를 스킬 실행과 기준 실행 양쪽에 동일하게 제공했다. 이는 오류 유도용 정보가 아니라 원자료 공백과 승인 범위를 명시한 실제 사용자 조건이다.

### 구현에서 반영한 수정

- brainstorm이 핵심 선택을 바꿀 의도·자료 누락을 구체적으로 질문하고, 확정된 조건을 다시 묻지 않도록 했다.
- `brief.md`, `materials.md`, `state.md`의 버전·승인·체크 계약을 공통 참조에 두고, brainstorm이 해당 절을 읽고 실제 기록을 남기게 했다.
- 내부 후보와 사용자 승인, 관측과 인과, 수행된 결과와 미래 계획의 경계를 루브릭과 공통 gate에 명시했다.

### 스킬 실행

- 모델·추론: `gpt-6-astra`, high, 새 문맥
- 제공 조건: B-1 원자료·요청, `report_writing/skills/brainstorm/SKILL.md`와 그 공통 참조. 강조점 후속 U5와 원자료 공백·승인 범위를 정한 U7은 기준 실행에도 동일하게 제공
- 응답 원문: `.work/validation/B-1/with-skill/response.md`, `response-followup.md`, `response-scope.md`
- 생성물: `.work/validation/B-1/with-skill/work/brief.md`, `materials.md`, `state.md`

최초 응답은 기능 변경 중심 방향을 승인안이 아닌 추천으로 표시하고, 제품 책임자가 먼저 이해·판단할 대상을 질문했다. U5로 구현 중심을 확정한 뒤에는 방·시간 통합의 직접 이유가 원자료에 없음을 발견해 이를 추측하지 않고 질문했다. U7은 새 사실을 주장하지 않고 통합 동기를 범위에서 제외했으며, 확인된 화면 구성 변화와 최종 확인 유지 이유를 중심으로 정했다. 관측 수치, A→B 고정 순서에 따른 인과 해석 제한, 추가 평가의 미수행 상태도 유지했다.

`brief.md`, `materials.md`, `state.md`는 모두 v0.3과 선행 입력·U1·U5·U7을 연결한다. 자료 풀은 M1~M4의 원문 위치와 참조 이유, M4의 미채택 후보 상태, U6 자료 공백과 U7의 범위 제외 해소를 보존한다. state는 B1·B2·B3·ALL-G12 각각의 결과·확인 위치·이유를 기록하며, 전체 새 제안은 부분 승인이고 spec 진행은 미승인임을 명시한다. spec과 HTML은 만들지 않았다.

판정: B1, B2, B3, ALL-G12와 인계·승인 경계를 모두 충족했다. B-1 행동 검증 통과. 관찰된 행동이 계약을 충족하여 추가 production 수정이나 재실행은 필요하지 않았다.

비차단 관찰사항이 두 가지 있다. brief는 최종 매체가 사용자에게 지정되지 않았다고 기록했는데, 패키지의 최종 산출물 기본값은 HTML이다. 이는 현재 요청이 작성안까지만 요구한 상황에서 잘못된 진행으로 이어지지 않았다. materials의 U2·U3 행은 당시 상태를 보존하고 해소를 후속 U5·U7 행에서 기록했다. 이력 보존 방식이며 이 사례에서는 현재 상태 오판이나 잘못된 진행이 관찰되지 않았다.

## S-1 · spec

### 스킬 없는 기준 실행

- 모델·추론: `gpt-6-astra`, high, 새 문맥
- 제공 조건: S-1 요청, 승인된 시작 brief의 조건과 큰 흐름, `tests/fixtures/kiosk/materials.md`의 원자료만 제공. 대상 스킬·구현 계획·설계 스펙·루브릭·정답은 미제공
- 원문: `.work/validation/S-1/control/response.md`
- 생성물: `.work/validation/S-1/control/spec.md`

기준 실행은 기능 변경의 선택 이유와 구현, 관측 결과와 한계, 후속 평가를 세 제목으로 분리하고 승인된 순서로 제시했다. 관측 핵심에는 성인 12명의 동일 과제, A→B 고정 순서, 각 1회 수행, 완료 시간·성공 관측값과 인과·일반화 한계를 함께 보존했다. 후속 평가는 아직 수행하지 않은 계획으로 구별했다. 이 핵심 내용과 간결성은 충족이다.

참조는 `M1`·`M2`·`M3` ID만 남겨 원문 locator와 참고 이유가 없었다. 생성물에는 spec 버전·선행 brief 버전·승인 상태, S별 S-G3~S-G5와 ALL-G12의 확인 위치·근거, 승인·진행 대응을 보존하는 state가 없었다. 이는 정확한 스펙 내용의 실패가 아니라 자료 접근과 지속 가능한 인계 계약의 차이다.

### 구현에서 반영한 수정

- spec이 모든 의미상 S에 제목 하나·핵심 하나·짧은 우선 참조를 배정하고, 성립 범위를 좌우하는 조건을 핵심 가까이에 보이게 했다.
- 우선 참조를 쓰기 전에 연결된 원자료를 확인하고 locator와 참조 이유를 남기며, 상세 후보·점검 기록은 사용자 검토용 스펙과 구별해 work에 보존하도록 했다.
- 모든 S의 S-G3~S-G5와 전체 ALL-G12를 state에 기록하고, 단계 전체를 한 번에 제시한 뒤 내용·구조·진행 gate를 적용하도록 했다.
- 설치 후에도 읽을 수 있는 연결 예시에 목표와 실제 설명, 소제목 소속, 변경 수준, 표현 방식, 경계 수정 사례를 자족적으로 담았다.

### 스킬 실행

- 모델·추론: `gpt-6-astra`, high, 새 문맥
- 제공 조건: S-1 원자료·승인된 시작 brief·요청, `report_writing/skills/spec/SKILL.md`와 그 공통 참조. 기준 실행과 같은 사용자 조건을 제공
- 응답 원문: `.work/validation/S-1/with-skill/response.md`
- 생성물: `.work/validation/S-1/with-skill/work/spec.md`, `materials.md`, `spec-notes.md`, `state.md`

전체 스펙은 승인된 이유→구현→관측과 한계→후속 평가 흐름을 S1~S4로 구성했다. S1은 최종 확인 유지 이유와 정량 검증 부재, S2는 두 화면 구현, S3는 12명의 A→B 고정 순서 관측과 인과 해석 제한, S4는 아직 수행하지 않은 후속 평가를 각각 하나의 목표로 맡았다. 요청은 고정 섹션 수를 정하지 않았으며 네 섹션은 독립 목표와 승인된 순서에 대응한다.

각 우선 참조에는 `input.md` 경로, M 제목 locator, 참고 이유가 함께 있고, 상세 설명 후보는 `spec-notes.md`로 분리했다. state에는 네 S 각각의 S-G3·S-G4·S-G5 12개 행과 전체 ALL-G12가 확인 위치·근거·필요한 수정과 함께 기록되었다. 시작 brief의 내용·구조와 spec 진행 승인은 현재 사용자 발언에 연결해 보존하고, 새 spec은 검토 후보와 미승인 상태로 두었다. writing-plan 진행을 확정하지 않았고 섹션별 승인을 요구하지 않았다.

판정: S-G3, S-G4, S-G5, ALL-G12와 인계·승인 경계를 모두 충족했다. S-1 행동 검증 통과. 관찰된 실패가 없어 production 수정과 행동 재실행은 필요하지 않았다.

## 형식 검사

다음 명령을 구현 파일 작성 후 실행했다.

```powershell
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/skill-creator/scripts/quick_validate.py report_writing/skills/brainstorm
& ./.venv/Scripts/python.exe -B -X utf8 C:/Users/minwoo/.codex-lab/skills/.system/plugin-creator/scripts/validate_plugin.py ./report_writing
git diff --cached --check
```

- 공식 skill 검사: `Skill is valid!` (exit 0)
- 공식 plugin 검사: `Plugin validation passed` (exit 0). 현재 구현된 brainstorm 단계의 패키지 구조 검사이며 네 단계 전체 판정은 Task 5에서 수행한다.
- 스테이징된 신규 파일 포함 공백 검사: 출력 없음 (exit 0)
