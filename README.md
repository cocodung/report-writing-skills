# Report Writing Skill

자료·기존 초안·작성 조건을 받아 글감 브레인스토밍 → 스펙시트 → 라이팅 플랜 → 원고 작성으로 이어지는 스킬을 개발한다. 보고서 전반에 적용하며 첫 적용 대상은 결과보고서다.

## 현재 개발 상태

2026-09-10 기준이다. 규칙과 승인 범위의 기준은 [설계 스펙 §6](docs/superpowers/specs/2026-09-09-report-writing-skill-design.md)이며, 이 문서는 개발 상태와 파일 위치를 안내한다.

| 항목 | 상태 |
|---|---|
| 개발 저장소 | `report-writing-skill/`를 별도 로컬 Git 저장소로 구성 |
| 스펙 정리 | 반복 규칙을 통합하고 형식별 예시·판정 사례를 보조 문서로 이관. [대응표와 검토 기록](docs/superpowers/specs/2026-09-09-integrated-distiller-writing-example-notes.md#spec-refactor-audit) |
| 설계 결정 | 그룹 6·7과 완료·인계·진입·재개를 포함한 세부 기준의 설계 검토 완료 |
| 호출·산출물 | `report_writing:brainstorm`, `report_writing:spec`, `report_writing:writing-plan`, `report_writing:develop` / 최종 HTML 보고서 |
| 첫 예시 | 원고 방향에 긍정적 평가. 해설 후 U·M·P·S 표기 유지 |
| 구현 계획 | [구현 계획 작성 완료·검토 대기](docs/superpowers/plans/2026-09-10-report-writing-skills-implementation.md). 실제 구현 전 |
| 실제 스킬 | SKILL.md 구현·설치 전 |

## 먼저 읽을 문서

1. [기준 설계 스펙](docs/superpowers/specs/2026-09-09-report-writing-skill-design.md): §1은 호출 이름·HTML 산출물·기본 저장 구성, §2는 네 단계, §3은 승인·점검·변경·재개, §4는 작성 기준, §5~§6은 구현 계획 인계와 검토 완료 상태를 다룬다.
2. [통합 증류기 연결 예시](docs/superpowers/specs/2026-09-09-integrated-distiller-writing-example.md): 자료 풀에서 실제 원고까지의 연결. §4 끝에 동일한 내용의 결과보고서형·논문형 비교를 둔다.
3. [예시 작업 메모](docs/superpowers/specs/2026-09-09-integrated-distiller-writing-example-notes.md): 자료 대조·선택 이유, §3의 판정·수정 사례, §5~§6의 제안·피드백·진행 이력, §7의 스펙 정리 기록.
4. [과거 비판적 검토](docs/superpowers/specs/2026-09-09-report-writing-skill-critical-review.md): 당시 스펙의 모호함과 보완 제안. 역사 기록으로 보존한다.

기준 스펙은 첫 번째 문서 하나다. 보조 문서를 읽을 조건과 각 문서의 권위는 스펙 §5에 정의한다. 기록된 사용자 동의는 그 내용과 범위에 적용되며 전체 스킬의 최종 승인으로 확대하지 않는다.

## 개발 폴더와 Git

기존 보고서 작업 폴더에서 스킬 개발을 분리하고 네 설계 문서를 이 저장소로 이동했다. 보고서·실험 자료는 상위 폴더에 둔다. 현재 대화를 이어가며 작업 경로만 이 저장소로 지정할 수 있다.

- `docs/superpowers/specs/`: 기준 스펙과 예시·작업 메모·과거 검토.
- `docs/superpowers/plans/`: 네 단계 스킬의 구현 계획.
- `skills/`: 현재 빈 자리표시 폴더. 구현 계획은 `report_writing/skills/`와 패키지 내 공통 참조로 구체화했다. 실제 파일 생성·경로 정리는 구현 시 진행한다.
- 저장소 안에서는 `git status`, `git log --oneline`으로 상태·이력을 확인한다. 상위 폴더에서는 `git -C .\report-writing-skill status`로 지정한다.

개발 폴더 구성은 스킬 실행 시 생성할 자료·산출물의 저장 구성과 구별한다. 예전 합의 경과와 정리 전 전체 스펙은 Git 이력에 보존한다.

## 다음 작업

1. [구현 계획](docs/superpowers/plans/2026-09-10-report-writing-skills-implementation.md)의 §1 파일 구성·§2 단계 간 계약과 Task 1~5를 검토한다. 이미 검토한 설계 세부 기준의 승인을 반복하지 않는다.
2. 구현 계획 검토 후 `report_writing/` 패키지의 네 단계 스킬과 공통 참조를 구현·검증한다. 최종 산출물은 HTML이며, 실제 설치·호출 발견 여부는 로컬 구현·검증과 구별해 기록한다.

기본 최종 경로는 `reports/<report-id>/report.html`이며, 중간 Markdown 자료와 진행 기록은 같은 문서 폴더의 `work/`에 둔다. 사용자 지정 경로와 재개 중인 기존 경로를 우선한다.

## 외부 원자료

- [현재 사용자 피드백](../리포트%20초안%20피드백0909.md)
- [현재 보고서 원고](../report_final.md)
- [이전 초안 피드백](../report_draft_feedback.md)
- [실험 코드 저장소](../Distillation_Project/)

원자료는 복사하지 않고 위치를 연결한다. 저장소를 다른 위치에 복제하면 원자료의 참조 위치를 다시 연결해야 한다.
