# Report Writing Skill

자료·기존 초안·작성 조건에서 시작해 글감 브레인스토밍 → 스펙 → 라이팅 플랜 → HTML 원고로 이어지는 네 단계 로컬 Codex 플러그인이다. 사용자가 단계 이름으로 진입하면 같은 보고서의 승인 상태와 작업 기록을 다음 단계로 넘긴다.

## 현재 개발 상태

2026-09-10 기준이다. 규칙과 승인 범위의 기준은 [설계 스펙 §6](docs/superpowers/specs/2026-09-09-report-writing-skill-design.md)이며, 이 문서는 개발 상태와 파일 위치를 안내한다.

| 항목 | 상태 |
|---|---|
| 개발 저장소 | `report-writing-skill/`를 별도 로컬 Git 저장소로 구성 |
| 설계 | 네 단계, 승인·재개·변경, 그룹 6·7, HTML 산출물 기준 검토 완료 |
| 구현 | `report_writing/skills/`의 네 SKILL.md와 공통 참조·HTML 자산 구현 완료 |
| 형식·패키지 검사 | 공식 네 skill 검사와 plugin 검사, 다른 임시 위치의 상대 참조·수동 호출 정책 검사 통과 |
| 행동 검사 | B/S/P/D 독립 사례와 I-1~I-7 통합 12개 사례 통과. 초기 P-1 그림 단계 실패와 수정·재실행도 기록 |
| HTML 확인 | 결과보고서형·논문형·재개·변경 사례의 정적 구조, 기록된 넓고 좁은 화면, 내부 링크 이동 확인 |
| 실제 설치 (Codex) | 개인 플러그인 `report_writing@personal` 설치·활성화 완료. 원래 보고서 폴더에서 네 스킬이 사용자 범위로 발견됨을 App Server로 확인 |
| 실제 설치 (Claude Code) | `.claude-plugin/` 매니페스트(플러그인·마켓플레이스) 추가 완료. 실제 설치·발견 검증은 대기 중 |

자세한 실행 조건과 근거는 [구현 검증 기록](docs/validation/implementation-validation.md)과 [설치 검증 기록](docs/validation/installation-validation.md)에 있다. 로컬 파일을 직접 읽힌 행동 평가와 설치 후 스킬 발견 검증을 구별한다.

## 네 단계 사용

전체 `report_writing/` 플러그인 루트를 함께 둔다. 네 단계가 공통 workflow·산출물 계약과 승인 state를 상대 경로로 공유하고 develop이 작성 기준·연결 예시·HTML 자산을 읽기 때문이다. 자동 호출은 꺼져 있으므로 아래 이름을 명시해서 진입한다. 한 문서를 계속 작성할 때는 단계 전체 승인과 진행 승인을 받으면 현재 에이전트가 다음 SKILL.md를 읽어 이어 가므로 단계마다 이름을 다시 부를 필요는 없다.

| 호출 | 입력 | 결과 |
|---|---|---|
| `report_writing:brainstorm` | 원자료·기존 초안·작성 조건과 핵심 선택 | `work/brief.md`, `materials.md`, `state.md` |
| `report_writing:spec` | 승인된 brief와 연결 자료 | 제목별 핵심·참조를 담은 `work/spec.md`와 갱신 state |
| `report_writing:writing-plan` | 승인된 spec·brief·materials | 실제 설명·근거·순서의 `work/writing-plan.md`, 필요한 경우 핵심만 담은 `figures.md` |
| `report_writing:develop` | 승인된 writing-plan과 작업 기록 | 브라우저용 `report.html`, 갱신 state와 필요한 전체 그림 메모 |

기본 최종 경로는 `reports/<report-id>/report.html`이다. 중간 Markdown과 승인·체크·변경·재개 기록은 같은 문서 루트의 `work/`에 둔다. 사용자가 지정한 경로와 재개 중인 기존 경로가 기본값보다 우선한다.

예를 들어 결과보고서형으로 시작할 때는 다음처럼 요청한다.

> `report_writing:brainstorm` 이 자료로 제품 책임자용 개발 결과보고서를 준비해줘. 가독성을 중시하고 구현 선택과 관측 한계를 연결해줘.

논문형이 필수라면 형식 조건을 함께 준다.

> `report_writing:brainstorm` 이 자료로 논문형 보고서를 준비해줘. 문단 중심 형식 안에서 가독성을 높여줘.

이미 승인한 플랜에서는 앞 단계를 되풀이하지 않고 현재 승인 발언으로 재개할 수 있다.

> 이 플랜은 승인했으니 `report_writing:develop`으로 이어서 써줘.

## Claude Code 설치

같은 `report_writing/` 배포 단위를 Claude Code 플러그인으로도 설치할 수 있다. 이 저장소가 프라이빗 마켓플레이스 역할을 하며, 플러그인 루트는 `report_writing/`으로 Codex와 동일하다 (`.codex-plugin/plugin.json` 옆에 `.claude-plugin/plugin.json`이 나란히 있다).

```
/plugin marketplace add cocodung/report-writing-skills
/plugin install report_writing@report-writing-skills
```

프라이빗 저장소이므로 GitHub 인증(브라우저 로그인 또는 자격 증명 관리자에 저장된 크리덴셜)이 필요하다. 설치 후 `report_writing:brainstorm` 등 네 이름이 `Skill` 도구 목록에 뜨는지 확인한다. 자동 호출은 꺼져 있으므로(각 SKILL.md의 description이 명시적 호출만 지시) 이름을 직접 불러 진입한다.

## 먼저 읽을 문서

1. [기준 설계 스펙](docs/superpowers/specs/2026-09-09-report-writing-skill-design.md): §1은 호출 이름·HTML 산출물·기본 저장 구성, §2는 네 단계, §3은 승인·점검·변경·재개, §4는 작성 기준, §5~§6은 문서 권위와 구현 상태를 다룬다.
2. [통합 증류기 연결 예시](docs/superpowers/specs/2026-09-09-integrated-distiller-writing-example.md): 자료 풀에서 실제 원고까지의 연결. §4 끝에 동일한 내용의 결과보고서형·논문형 비교를 둔다.
3. [예시 작업 메모](docs/superpowers/specs/2026-09-09-integrated-distiller-writing-example-notes.md): 자료 대조·선택 이유, §3의 판정·수정 사례, §5~§6의 제안·피드백·진행 이력, §7의 스펙 정리 기록.
4. [과거 비판적 검토](docs/superpowers/specs/2026-09-09-report-writing-skill-critical-review.md): 당시 스펙의 모호함과 보완 제안. 역사 기록으로 보존한다.

기준 스펙은 첫 번째 문서 하나다. 보조 문서를 읽을 조건과 각 문서의 권위는 스펙 §5에 정의한다. 기록된 사용자 동의는 그 내용과 범위에 적용되며 전체 스킬의 최종 승인으로 확대하지 않는다.

## 개발 폴더와 Git

기존 보고서 작업 폴더에서 스킬 개발을 분리하고 네 설계 문서를 이 저장소로 이동했다. 보고서·실험 자료는 상위 폴더에 둔다. 현재 대화를 이어가며 작업 경로만 이 저장소로 지정할 수 있다.

- `docs/superpowers/specs/`: 기준 스펙과 예시·작업 메모·과거 검토.
- `docs/superpowers/plans/`: 네 단계 스킬의 구현 계획.
- `report_writing/skills/`: 배포 단위 안의 단계별 스킬. 공통 진행·산출물 계약은 `report_writing/references/`에 둔다.
- 저장소 안에서는 `git status`, `git log --oneline`으로 상태·이력을 확인한다. 상위 폴더에서는 `git -C .\report-writing-skill status`로 지정한다.

개발 폴더 구성은 스킬 실행 시 생성할 자료·산출물의 저장 구성과 구별한다. 예전 합의 경과와 정리 전 전체 스펙은 Git 이력에 보존한다.

## 다음 작업

로컬 구현과 계획된 검증에 이어, 사용자의 후속 지시로 개인 플러그인 설치와 네 호출 이름의 발견 확인까지 완료했다. 원래 보고서 폴더에서 새 Codex 대화를 열어 `$report_writing:brainstorm` 등으로 시작한다. 개발 브랜치는 `feat/report-writing-skills`에 보존했으며 워크트리를 옮기거나 main에 병합할 필요는 없다.

설치 원본은 `C:/Users/minwoo/plugins/report_writing/`, 현재 설치본은 `C:/Users/minwoo/.codex-lab/plugins/cache/personal/report_writing/0.1.0/`에 있다. 개발 소스를 수정해도 설치본이 자동 갱신되지는 않으므로, 이후에는 설치 원본을 갱신하고 재설치한다. 경로와 갱신 절차는 설치 검증 기록에 정리했다.

## 외부 원자료

- [현재 사용자 피드백](../리포트%20초안%20피드백0909.md)
- [현재 보고서 원고](../report_final.md)
- [이전 초안 피드백](../report_draft_feedback.md)
- [실험 코드 저장소](../Distillation_Project/)

원자료는 복사하지 않고 위치를 연결한다. 저장소를 다른 위치에 복제하면 원자료의 참조 위치를 다시 연결해야 한다.
