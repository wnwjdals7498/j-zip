# PMT 운영 규칙

이 폴더(`work/pmt/`)는 j-zip의 진행 상태, 재개 정보, 차단 관계, 운영 결정, 검증 기록을 PMT(Project Management Tool, `proj-mgmt-tool` 스킬의 `scripts/pmt.py`)로 관리하는 데이터 루트다.
PMT의 기본 데이터 루트는 `~/docs`지만, j-zip은 사용자 결정(2026-11-05)에 따라 **이 레포 안에서만** 관리한다. 레포 밖(`~/docs/projects/`)에 j-zip 프로젝트를 만들지 않는다.

## 1. 원천 구분

한 가지 내용은 한 곳에만 원천을 둔다. 다른 곳은 그 원천을 가리킨다.

| 내용 | 원천 | PMT에서의 처리 |
|---|---|---|
| 작업 정의(설명, 근거, 선행, 크기, 필수 증거) | [docs/11](../../docs/11-implementation-backlog.md) | Item `## 기본 내용`에 11 행을 옮겨 둔다. 11이 바뀌면 11을 먼저 고치고 같은 날 Item을 맞춘다 |
| 구현자에게 넘기는 작업 명세 | `work/tasks/<작업 ID>.md` ([카드 표준](../tasks/README.md)) | Item `## 범위`에 카드 작성·검수 단계로만 적는다 |
| 진행 상태, 재개, 차단, 검증 기록 | **이 폴더** | `start`·`note`·`verify`·`end`·`set` 명령으로만 바꾼다 |
| 구현·설계 결정 | `work/decisions/` (ADR) | `pmt decide`를 쓰지 않는다. 채택된 ADR은 `pmt add fact "ADR <이름> 채택" --ref work/decisions/<파일>`로 가리킨다 |
| 프로젝트 운영 결정(진행 방식, 범위, 우선순위, 도구) | **이 폴더** `decisions.md` | `pmt decide`로 기록한다 |
| 증거 | `work/evidence/<작업 ID>/` ([docs/10](../../docs/10-logging-evidence-operations.md) 3절) | Item `## 증거`에 경로만 적는다 |

## 2. 호출

PMT는 **항상 main 체크아웃의 `work/pmt/`** 를 데이터 루트로 쓴다. 작업 worktree 안에서 실행해도 같은 폴더를 가리키도록 git 공통 디렉터리에서 경로를 계산한다. worktree 안에 있는 `work/pmt/` 사본은 고치지 않는다.

```bash
# bash (레포 루트 또는 아무 worktree 안에서)
export PMT_DOCS_ROOT="$(git rev-parse --path-format=absolute --git-common-dir)/../work/pmt"
export PMT_SESSION="<세션마다 고정한 ID>"
python3 <skill-dir>/scripts/pmt.py --project j-zip resume j-zip
```

```powershell
# PowerShell
$env:PMT_DOCS_ROOT = Join-Path (Split-Path (git rev-parse --path-format=absolute --git-common-dir)) 'work/pmt'
$env:PMT_SESSION = '<세션마다 고정한 ID>'
python <skill-dir>/scripts/pmt.py --project j-zip resume j-zip
```

- `<skill-dir>`은 `proj-mgmt-tool` 스킬 폴더다. 환경별 경로는 스킬의 `references/installation.md`를 따른다.
- `PMT_DOCS_ROOT`(또는 `--docs-root`)를 빠뜨리면 기록이 `~/docs`로 간다. 명령 실행 전에 확인한다.
- 같은 세션의 `start`와 `end`는 같은 `PMT_SESSION`으로 호출한다. lock이 세션에 묶이기 때문이다.
- `project.md`의 `repositories: [.]`는 "명령을 실행한 위치의 git 저장소"라는 뜻이다. `verify`는 검증할 작업의 worktree 안에서 실행해 그 HEAD가 기록되게 한다.
- 콜드스타트나 컨텍스트 압축 뒤에는 `resume j-zip` 출력과 작업할 Item 파일 하나만 읽고 시작한다.

## 3. 구조

| 경로 | 내용 |
|---|---|
| `projects/j-zip/project.md` | Goal, Non-Goal, 원천 문서 |
| `projects/j-zip/RESUME.md` | 재개 요약(엔진 생성물, 손으로 고치지 않음) |
| `projects/j-zip/_default/Work1*.md` | 기획 문서 정리: 이 운영 체계 구축, 미결 분류, 초안 카드 검수, PMT 커밋 경로 결정 |
| `projects/j-zip/G<n>/Work1.md` | 단계 G0–G6 하나당 Work 하나. Goal은 그 단계의 종료 조건([docs/09](../../docs/09-development-process.md) 3절) |
| `projects/j-zip/G<n>/Work1-<m>.md` | 11의 작업 하나당 Item 하나. 제목은 11 작업 ID로 시작하고 11의 행 순서대로 번호를 붙였다 |
| `projects/j-zip/{facts,decisions,backlog}.md` | 사실, 운영 결정, 백로그 목록 |
| `worklog/`, `times/` | 엔진이 남기는 작업 기록과 시간 기록 |

- 작업 ID로 Item을 찾는다: `pmt --project j-zip find G2-F17-T03`.
- Item의 `blocked_by`는 11의 선행 열을 그대로 옮긴 것이다(`G<n>의 다른 모든 작업`은 그 단계의 나머지 Item 전부). Item을 `end --done`으로 끝내면 엔진이 후행 Item의 `blocked_by`에서 지우고, 차단이 모두 풀린 Item이 `RESUME.md`의 "착수 가능"에 나온다.
- 11에 없는 차단 관계: `G0-INFRA-T16`(`G0/Work1-1`)은 `_default/Work1-4`(PMT 커밋 경로 결정)에 막혀 있다.
- 11에 새 작업 ID가 생기면(L 작업 분할 등) 11에 먼저 추가한 뒤 `pmt add item j-zip/G<n>/Work1 "<작업 ID> <제목>"`으로 등록하고 `blocked_by`를 11과 맞춘다.

## 4. 카드 상태와 PMT 상태

| 카드 상태([카드 표준](../tasks/README.md) 7절) | PMT Item | 명령 |
|---|---|---|
| 카드 없음, 초안, 준비됨 | Planned | — |
| 진행, 검토 | In Progress | `start <item-id>` (핸드오프 직전) |
| 보류 | In Progress 유지 + 정지 사유 기록 | `end <item-id> --pause --did "..." --next "..."` 또는 `set <item-id> --blocked-by ext:<사유>` |
| 완료 | Done | `verify` 뒤 `end <item-id> --done --result "..."` |

- 카드 상태와 PMT 상태는 같은 날 맞춘다. 둘이 다르면 카드 상태가 맞는 것으로 보고 PMT를 고친다.
- Item 상태는 명령으로만 바꾼다. Item 본문의 `## 기본 내용`·`## 범위`·`## 완료 기준`은 직접 고칠 수 있다.

## 5. 쓰는 주체와 커밋

- `work/pmt/**`는 오케스트레이터만 쓴다([docs/12](../../docs/12-agent-orchestration.md) 2절). 외부 구현·검토 모델과 서브에이전트는 이 폴더를 읽기만 한다.
- lock(`projects/*/.locks/`, `.repo-locks/`)은 커밋하지 않는다(`.gitignore`).
- 커밋 경로(PMT 결정 D1, 2026-11-05): PMT 변경은 오케스트레이터가 **main 체크아웃에서 `main`에 기록 전용 커밋으로 직접 커밋**한다. 작업 브랜치·PR에 넣지 않는다.
  - 기록 전용 커밋은 `work/pmt/**`만 바꾼다. 코드·문서 변경과 섞지 않는다([docs/09](../../docs/09-development-process.md) 7절). 이 커밋은 CI 경로 필터가 빌드·테스트를 건너뛴다.
  - 커밋 시점: `start`·`end`·`decide`·`set`으로 상태가 바뀐 직후. `note`만 바뀐 것은 다음 상태 변경 커밋에 함께 넣어도 된다.
  - 커밋 메시지: `chore(pmt): <요약> [<작업 ID 또는 PMT Item ID>]`.
  - push는 `main` 직접 push 금지의 예외다([docs/09](../../docs/09-development-process.md) 7절). push 전에 `git pull --rebase`로 원격 `main`(병합된 PR)을 먼저 받는다. 강제 push는 금지한다.
