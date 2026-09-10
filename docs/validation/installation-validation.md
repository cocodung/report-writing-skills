# 개인 플러그인 설치 검증

2026-09-10, 사용자가 개발 브랜치를 보존한 채 원래 보고서 폴더에서 사용할 수 있도록 설치를 승인했다. 구현 패키지 기준은 `c45f0f5`, 버전은 `0.1.0`이다. main 병합이나 외부 배포는 하지 않았다.

## 실제 경로

| 역할 | 경로 |
|---|---|
| 개발 소스 | `report-writing-skill/.worktrees/implementation/report_writing/` |
| 개발 브랜치 | `feat/report-writing-skills` |
| 개인 마켓플레이스 | `C:/Users/minwoo/.agents/plugins/marketplace.json` |
| 설치용 원본 | `C:/Users/minwoo/plugins/report_writing/` |
| 설치 캐시 | `C:/Users/minwoo/.codex-lab/plugins/cache/personal/report_writing/0.1.0/` |
| 발견 확인의 작업 폴더 | `C:/Users/minwoo/Documents/coop보고서제작` |

현재 셸의 `CODEX_HOME`은 `.codex-lab`이다. 기본 경로에서 발견된 CLI는 `C:/Users/minwoo/AppData/Local/Programs/OpenAI/Codex/bin/codex.exe`이며 이 사용자 환경에 설치했다.

## 설치와 확인

공식 `create_basic_plugin.py`의 마켓플레이스 작성 함수를 사용해 개인 항목을 추가했다. CLI scaffold의 이름 정규화는 `_`를 `-`로 바꾸므로, 기존에 승인·검증한 `report_writing` 식별자를 유지하는 helper 함수를 직접 사용했다. 항목은 `source.source=local`, `source.path=./plugins/report_writing`, `installation=AVAILABLE`, `authentication=ON_INSTALL`, `category=Productivity`이다.

최초 설치는 개인 source 경로를 `~/.agents/plugins/plugins/report_writing`에 배치해 실패했다. 실제 CLI 오류는 `~/plugins/report_writing`을 요구했다. 그 실제 경로로 동일한 파일 14개를 복사한 뒤 성공했으며 미사용 최초 복사본은 경로·내용 검증 후 정리했다. 마켓플레이스의 상대 경로는 바꾸지 않았다.

```powershell
codex plugin add report_writing@personal
codex plugin list --json
```

설치 명령은 exit 0과 설치 캐시 경로를 반환했다. 목록에서 `pluginId=report_writing@personal`, `version=0.1.0`, `installed=true`, `enabled=true`를 확인했다. 원격 카탈로그 조회 경고는 발생했지만 로컬 설치 항목과 다음 직접 발견 결과를 확인했다.

Codex App Server를 stdio로 실행하여 `initialize`/`initialized` 후 다음 요청을 보냈다. 보고서 생성이나 모델 turn은 시작하지 않았다.

```json
{"id":1,"method":"skills/list","params":{"cwds":["C:/Users/minwoo/Documents/coop보고서제작"],"forceReload":true}}
```

| 발견된 이름 | 범위 | 활성화 |
|---|---|---|
| `report_writing:brainstorm` | user | true |
| `report_writing:spec` | user | true |
| `report_writing:writing-plan` | user | true |
| `report_writing:develop` | user | true |

네 경로 모두 위 설치 캐시를 가리켰고 응답의 `errors`는 빈 배열이었다. 설치 캐시의 파일 14개는 개발 소스와 바이트 단위로 같았다. 네 `agents/openai.yaml`의 `allow_implicit_invocation=false`를 직접 확인했으며 공식 `validate_plugin.py`의 설치 캐시 검사도 통과했다.

이 검증은 실제 설치·활성화·작업 폴더별 스킬 발견과 파일 대응을 확인한 것이다. 새 VS Code 대화에서 UI를 클릭하거나 설치된 스킬로 보고서를 새로 작성한 검증은 아니다. 이전 행동 평가 결과는 구현 검증 기록에 보존되어 있다. 필터링한 발견 응답과 검사 도구는 Git 제외 경로 `.work/installation/`에 있다.

## 사용과 이후 갱신

원래 보고서 폴더를 연 채 새 Codex 대화에서 `$report_writing:brainstorm` 등으로 시작한다. 개발 워크트리나 main 병합은 실행 조건이 아니다. 현재 대화의 스킬 목록은 이전에 구성됐으므로 새 대화에서 설치 상태를 반영한다.

개발 소스 수정은 설치 캐시로 자동 전달되지 않는다. 갱신 시 승인된 전체 `report_writing/`를 설치용 원본에 반영하고, 공식 `update_plugin_cachebuster.py`를 그 원본 경로에 적용한 다음 같은 사용자 환경에서 `codex plugin add report_writing@personal`을 실행한다. 새 대화에서 변경 버전의 발견과 수동 호출을 확인한다. 설치용 버전 표시를 바꾸기 위해 개발 브랜치를 main에 병합할 필요는 없다.
