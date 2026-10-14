# 외부 코딩 에이전트 핸드오프 조사 (OMC 5.7.2 / Codex / Antigravity)

- 조사일: 2026-10-14
- 범위: 읽기 전용 조사. `--help`/`--version`/`agy models` 외에는 프롬프트를 실행하지 않았고 설정도 변경하지 않았다.
- 태그 규칙: `[확인]` = 이 머신에서 소스/`--help`/설정 파일로 직접 확인, `[미확인]` = 확인하지 못했거나 추정.

## 0. 요약 (TL;DR)

1. `omc ask`는 **모델 선택 플래그가 없다**. 프롬프트만 넘기고 각 CLI의 기본 모델을 쓴다. 지정 모델(`gpt-6-luna`, `gemini-3.8-flash-medium`)을 쓰려면 `codex exec -m ...` / `agy --model ...`를 직접 호출해야 한다. [확인]
2. `omc ask` SKILL은 "원시 CLI를 직접 실행하지 말라"고 하지만, 이는 `omc ask` 스킬 경로에 대한 규칙이다. 모델 고정이 필요한 j-zip 계획에서는 충돌하므로 **계획 문서에 예외로 명시**해야 한다. [확인]
3. `omc ask codex`/`antigravity`는 각각 `--dangerously-bypass-approvals-and-sandbox` / `--dangerously-skip-permissions`로 실행되어 **외부 에이전트가 저장소를 수정할 수 있다** (답변 전용이 아님). [확인]
4. 사용자가 말한 이름은 실제 슬러그와 약간 다르다: `gpt6-luna` -> `gpt-6-luna`, `gemini3.8flash-medium` -> `gemini-3.8-flash-medium`. [확인]
5. 이 머신에는 `tmux`/`psmux`가 없어 `omc team`(tmux 워커)은 사용 불가. [확인]

## 1. `omc ask` 동작 (질문 1)

### 1.1 문서/소스 위치
- 스킬 shim: `C:\Users\wnwjdals7498\.claude-company\plugins\cache\omc\oh-my-claudecode\5.7.2\skills\ask\SKILL.md` (본문은 `skill-bodies/ask/SKILL.md`로 위임) [확인]
- 마켓플레이스 사본: `plugins/marketplaces/` 하위에 `claude-plugins-official`, `omc`가 있으나 내용 대조는 하지 않음 [미확인]
- 구현: `...\5.7.2\src\cli\ask.ts`(인자 파싱), `...\5.7.2\scripts\run-provider-advisor.js`(실제 실행/아티팩트 기록) [확인]

### 1.2 정확한 명령 문법 [확인]
`omc ask --help` 출력:
```
omc ask <claude|codex|gemini|antigravity|grok|cursor|muse> <question or task>
omc ask <provider> -p "<prompt>"
omc ask <provider> --print "<prompt>"
omc ask <provider> --prompt "<prompt>"
omc ask <provider> --agent-prompt <role> "<prompt>"
```
- 스킬 호출 형태: `/oh-my-claudecode:ask codex "..."` -> 내부적으로 `omc ask codex "..."` 실행.
- 프롬프트는 **인자 문자열 하나**뿐이다. 파일 첨부/`--model`/`--cd`/`--sandbox` 옵션은 파서에 없고, 알 수 없는 토큰은 프롬프트 텍스트로 합쳐진다. [확인: `parseAskArgs`]
- `--agent-prompt <role>`: `agents/<role>.md`(또는 `$CODEX_HOME/prompts/<role>.md`)의 역할 프롬프트를 프롬프트 앞에 붙인다. [확인]
- 파일 내용을 넘기려면 프롬프트에 경로를 적어 외부 에이전트가 직접 읽게 하거나 내용을 인라인해야 한다. [확인: 파일 인자 옵션 없음]

### 1.3 프로바이더별 실제 실행 형태 [확인: `run-provider-advisor.js` `buildProviderArgs`]
| provider | 바이너리 | 실제 argv |
|---|---|---|
| codex | `codex` | `codex exec --dangerously-bypass-approvals-and-sandbox <prompt>` (프롬프트가 500자 초과/개행 포함/Windows 셸이면 `-`로 stdin 전달) |
| antigravity | `agy` | `agy --dangerously-skip-permissions -p <prompt>` (stdin 불가, argv 전용, 셸 없이 spawn) |
| gemini | `gemini` | `gemini -p <prompt> --yolo` |
| claude | `claude` | `claude -p <prompt>` |

- 모델 선택: **없음**. 위 argv 어디에도 `-m/--model`이 없다. 각 CLI 설정의 기본 모델이 쓰인다 (Codex는 `~/.codex/config.toml`의 `model`, agy는 CLI 기본값). [확인]
- 모델 환경변수(`OMC_CODEX_DEFAULT_MODEL`, `OMC_ANTIGRAVITY_DEFAULT_MODEL` 등)는 **`omc team` 워커 경로에서만** 읽힌다(`src/team/model-contract.ts`, `docs/REFERENCE.md` 120-122행). `omc ask` 코드(`ask.ts`, `run-provider-advisor.js`)에는 참조가 없다. [확인]

### 1.4 결과물/아티팩트 [확인]
- 경로: `<프로젝트>/.omc/artifacts/ask/<provider>-<slug>-<timestamp>.md` (`OMC_STATE_DIR` 설정 시 해당 루트 아래). `slug`는 원 프롬프트를 소문자 영숫자-하이픈 60자로 줄인 값, 타임스탬프는 ISO 문자열의 `:`/`.`를 `-`로 치환.
- 파일 구성: Provider, Exit code, Created at, `## Original task`, `## Final prompt`, `## Raw output`(코드 블록), `## Concise summary`(고정 문구), `## Action items`(고정 문구).
- `omc ask`는 **stdout에 아티팩트 경로를 출력**하고 종료한다. 오케스트레이터는 그 경로를 읽어 결과를 얻는다. 외부 에이전트가 만든 코드 변경은 아티팩트가 아니라 **작업 트리에 직접 반영**된다.
- j-zip의 `.gitignore`는 `.omc/`를 무시한다 -> 아티팩트는 커밋되지 않는다. 보존이 필요한 결과는 `docs/` 등으로 옮겨야 한다. [확인]
- 종료 코드: 래퍼는 provider 종료 코드를 그대로 전파(0이 아니면 비0). 실패해도 아티팩트는 기록된다. [확인]

### 1.5 타임아웃 [확인]
- codex: 래퍼 자체 타임아웃 없음(`spawnSync`에 timeout 미지정). 장시간 작업은 호출 측(Bash 도구 timeout 최대 600000ms, 또는 `run_in_background`)에서 관리해야 한다.
- antigravity: 기본 300000ms(5분), `OMC_ANTIGRAVITY_TIMEOUT_MS`로 조정(정수, 1000 이상, 상한 3600000). 초과 시 SIGKILL 후 실패 처리. `agy --print-timeout`은 업스트림 버그로 동작하지 않는다고 소스 주석에 명시(antigravity-cli#76). 종료 0이지만 출력이 비면 실패로 간주.
- 출력 버퍼 상한 10MB (`maxBuffer`).

### 1.6 권한/쓰기 [확인]
- codex: `--dangerously-bypass-approvals-and-sandbox` = 승인/샌드박스 전부 해제. 저장소 편집, 명령 실행 가능.
- antigravity: `--dangerously-skip-permissions` = 도구 권한 요청 자동 승인. 편집 가능.
- 즉 "답변만 하는 자문"이 아니라 **편집 권한이 있는 자율 에이전트**다. 읽기 전용 자문이 필요하면 `omc ask`로는 불가능하고 직접 `codex exec -s read-only` 등을 써야 한다.
- `disableExternalLLM` 보안 설정이 켜져 있으면 claude 외 provider는 차단된다(`ask.ts`). 이 머신의 `.omc-config.json`에는 해당 설정이 없다(`nodeBinary`만 존재). [확인]

### 1.7 알려진 불일치/주의
- SKILL 본문(`skill-bodies/ask/SKILL.md`)은 "Windows에서 `omc ask antigravity`는 가드되어 오류, Windows에서는 `omc ask gemini` 사용"이라고 적혀 있으나, 실제 스크립트 `guardProviderPlatform`은 "플랫폼별 차단 없음"이고 주석은 agy 1.0.15/1.1.1에서 Windows 문제가 수정되었다고 적는다. 문서가 오래된 것으로 보인다. [확인: 불일치 자체] 이 머신 agy는 1.3.5이므로 동작 가능성이 높지만 실제 실행은 하지 않았다. [미확인]
- `gemini` CLI는 이 머신에 없다. `omc ask gemini`는 `Missing required local CLI binary: gemini`로 실패한다. [확인]
- PowerShell에서 `omc`는 `omc.ps1`(npm shim)로 실행되며, 매 실행마다 "tmux 없음" 경고를 출력한다(무해). [확인]
- j-zip은 `.git`이 존재하는 git 저장소다. [확인] (`codex exec`의 git 저장소 검사를 통과한다.)

## 2. 설치된 CLI와 모델 선택 (질문 2)

| 도구 | 경로 / 버전 | 비고 |
|---|---|---|
| omc | `C:\nvm4w\nodejs\omc.ps1`, 5.7.2 | [확인] |
| codex | `C:\nvm4w\nodejs\codex.ps1`, codex-cli 0.156.1 | [확인] |
| agy | `C:\Users\wnwjdals7498\AppData\Local\agy\bin\agy.EXE`, 1.3.5 | [확인] |
| gemini | 없음 | [확인] |
| antigravity (별칭) | 없음 (`agy`만 존재) | [확인] |
| claude | `C:\Users\wnwjdals7498\.local\bin\claude.exe` | [확인] |
| tmux / psmux | 없음 | [확인] |

### 2.1 Codex 모델 선택 [확인]
- 플래그: `-m, --model <MODEL>` (`codex exec`에도 존재), 또는 `-c model="..."`, 프로필 `-p/--profile <name>`(`$CODEX_HOME/<name>.config.toml`을 기본 설정 위에 레이어).
- 이 머신 `~/.codex/config.toml` 기본값: `model = "gpt-6.1-sol"`, `model_reasoning_effort = "xhigh"`, `model_context_window = 1000000`. 따라서 `omc ask codex`는 지정 모델이 아닌 **`gpt-6.1-sol`(xhigh)을 사용**한다.
- 모델 카탈로그 캐시(`~/.codex/models_cache.json`)의 슬러그: `gpt-6-astra`, `gpt-6-sol`, **`gpt-6-luna`**, `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`, `gpt-5.5`(hide) 등. 사용자가 말한 `gpt6-luna`는 **`gpt-6-luna`** 로 매핑된다. 캐시 기준이며 계정에서 실제 호출 가능한지는 실행해 봐야 안다. [미확인]
- 추론 강도 지정: `-c model_reasoning_effort="medium"` 형태. [확인: omc 소스가 같은 형태 사용]
- `~/.codex/j-messenger.config.toml`은 다른 프로젝트용 프로필(freellmapi 프로바이더, localhost:3001)이므로 j-zip에서 `-p j-messenger`로 쓰지 말 것. [확인] j-zip 전용 프로필은 만들지 않았다(설정 변경 금지).

### 2.2 Antigravity(agy) 모델 선택 [확인]
- 플래그: `--model <id>`, 추론 강도 `--effort low|medium|high|xhigh|max`. `agy models`가 사용 가능한 모델을 나열한다(프롬프트 미전송).
- `agy models` 출력(이 머신): `gemini-3.8-flash-high`, **`gemini-3.8-flash-medium`**, `gemini-3.8-flash-low`, `claude-opus-5-5-{low,medium,high}`, `claude-sonnet-5-5-{low,medium,high}`, `gpt-oss-120b-medium`.
- 사용자가 말한 `gemini3.8flash-medium`은 **`gemini-3.8-flash-medium`** 으로 매핑된다. 모델 ID에 강도(medium)가 포함되어 있으므로 `--effort`는 따로 줄 필요가 없을 가능성이 크다. [미확인: `--effort`와 모델 ID 접미사의 상호작용]

### 2.3 omc 자체의 모델 설정
- `omc team`용: `OMC_CODEX_DEFAULT_MODEL`, `OMC_ANTIGRAVITY_DEFAULT_MODEL`, `OMC_EXTERNAL_MODELS_DEFAULT_<PROVIDER>_MODEL` 환경변수 또는 설정의 externalModels/team role routing. `omc ask`에는 적용 안 됨(1.3). 이 머신 셸에는 `OMC_*`/`CODEX*`/`ANTIGRAVITY*` 환경변수가 없다. [확인]

## 3. 팀/핸드오프 규칙 (질문 3)

### 3.1 `.omc/handoffs/` [확인: `skill-bodies/team/SKILL.md` 약 152-200행, 전역 `CLAUDE.md`]
- **스테이지 간 핸드오프 문서**(team-plan -> team-prd -> team-exec -> team-verify -> team-fix)이며, **외부 모델에 일을 넘기는 용도가 아니다**. 리드가 `.omc/handoffs/<stage-name>.md`에 작성한다.
- 형식(10-20줄 권장):
```markdown
## Handoff: <current-stage> → <next-stage>

- **Decided**: ...
- **Rejected**: ...
- **Risks**: ...
- **Files**: ...
- **Remaining**: ...
```
- 취소해도 삭제되지 않으며 재개 시 읽는다. 전역 `CLAUDE.md`는 재개/컴팩션 후 편집 전에 `.omc/handoffs/`를 다시 확인하라고 지시한다.

### 3.2 외부 모델 위임 경로 두 가지
1. **`omc ask <provider> "<prompt>"` (1회성 자문/작업)**: 결과는 `.omc/artifacts/ask/*.md`. 동기 실행, 경로를 stdout으로 반환. 팀 상태와 무관. [확인]
2. **`omc team N:codex|antigravity[:role] "<task>"` (tmux 워커)**: `omc team api`(`send-message` 등 JSON API)로 ACK/결과를 주고받고 `omc team status|shutdown <team>`으로 관리. 워커는 `working_directory`에서 자율 실행하며 결과를 출력 파일(예: `.omc/research/*.md`)에 기록. 모델은 `OMC_*_DEFAULT_MODEL` 등으로 지정 가능. 워커 실행 플래그: codex `--dangerously-bypass-approvals-and-sandbox [--model]`, agy `--dangerously-skip-permissions [--model] [--effort]`. **tmux 호환 바이너리가 필요한데 이 머신에는 없다** (omc 경고: `winget install psmux`). [확인] -> 현재 사용 불가.
- Claude Code 내부 팀(`/team`)은 Claude 에이전트용 네이티브 기능이며, 외부 CLI 워커는 Claude의 태스크 리스트/팀 메시징에 참여하지 못한다. [확인: team SKILL "Key difference" 항목]

### 3.3 기타
- `ultragoal`: `.omc/ultragoal/`에 계획/원장 아티팩트를 남기고 Claude `/goal`용 핸드오프 텍스트를 출력하는 **Claude 세션 전용** 워크플로. 외부 모델 위임 기능은 없다. [확인]
- `execute`/`ultragoal` 스킬 본문에서 외부 모델 위임 규칙은 찾지 못했다(grep codex/gemini/antigravity 결과 없음). [확인]
- 별도 표준 "외부 모델 결과 포맷"은 없다. `omc ask` 아티팩트 md(1.4)가 유일한 정형 포맷이다. 구조화 결과가 필요하면 프롬프트에서 형식을 지정하거나 Codex `--output-schema` / agy `--json-schema`를 쓴다.

## 4. Codex / Antigravity 비대화형 모드 일반 (질문 4)

### 4.1 Codex `codex exec` [확인: `codex exec --help` 0.156.1, 공식 문서]
- 프롬프트: 위치 인자, 또는 인자 없음/`-`이면 stdin. 프롬프트 인자와 stdin 파이프가 함께 있으면 stdin이 `<stdin>` 블록으로 추가됨.
- 주요 옵션:
  - `-m, --model <MODEL>`
  - `-s, --sandbox <read-only|workspace-write|danger-full-access>`
  - `-C, --cd <DIR>`: 작업 루트
  - `--add-dir <DIR>`: 추가로 쓰기 허용할 디렉터리
  - `--skip-git-repo-check`: git 저장소가 아닌 곳에서 실행 허용
  - `-o, --output-last-message <FILE>`: 마지막 메시지를 파일로 저장 (stdout에도 출력, 진행 로그는 stderr)
  - `--json`: JSONL 이벤트 스트림(`thread.started`, `turn.completed`, `item.*` 등)
  - `--output-schema <FILE>`: 최종 응답 JSON 스키마 강제
  - `--ephemeral`: 세션 파일 미저장
  - `-c key=value`: 설정 덮어쓰기 (예: `-c model_reasoning_effort="medium"`)
  - `--ignore-user-config`, `--ignore-rules`, `--worktree`, `--approve-for-me`
  - 서브명령: `codex exec resume`, `codex exec fork`, `codex exec review`
- 기본 샌드박스: **read-only** (공식 문서). 편집하려면 `--sandbox workspace-write`.
- `--full-auto`는 **더 이상 지원되지 않음** (공식 문서). 대체: `--sandbox workspace-write` 또는 `--approve-for-me`. 이 버전 `--help`에도 `--full-auto`가 없다. [확인]
- `-a/--ask-for-approval`은 최상위 `codex`에만 있고 `exec --help`에는 없다. [확인]
- 인증: 저장된 CLI 자격 증명 재사용(`~/.codex/auth.json` 존재 [확인]). CI에서는 `CODEX_API_KEY` 권장(공식 문서).
- Windows 샌드박스: 이 머신 config에 `[windows] sandbox = "elevated"`. `workspace-write`가 Windows에서 실제로 쓰기를 허용하는지는 실행해 봐야 안다. [미확인]
- 종료 코드: 공식 문서에 명시 없음(필수 MCP 서버 초기화 실패 시 오류 종료만 언급). 0/비0로만 판단하고 세부 값은 [미확인].
- 이 머신 기본값은 `gpt-6.1-sol`/`xhigh`/1M 컨텍스트라 느리고 비쌀 수 있음 -> `-m gpt-6-luna`와 `-c model_reasoning_effort=...`로 명시하는 것이 안전하다.
- Codex 훅(`~/.codex/hooks.json`)과 플러그인이 활성화되어 있어 `codex exec`에서도 훅이 돌 수 있다. 격리하려면 `--ignore-user-config`(인증은 유지)를 고려하되 영향 범위는 [미확인].

### 4.2 Antigravity `agy` [확인: `agy --help` 1.3.5]
- `-p/--print/--prompt "<prompt>"`: 단일 프롬프트 비대화형 실행 후 응답 출력. **프롬프트는 argv 값이며 stdin으로 못 읽는다** (omc 소스 주석; `--input-format stream-json`은 stdin NDJSON 별도 모드).
- 주요 옵션:
  - `--model <id>`, `--effort low|medium|high|xhigh|max`
  - `--mode accept-edits|auto|plan`: 실행 모드. `plan`은 계획 전용으로 보이나 정확한 의미는 [미확인]
  - `--dangerously-skip-permissions`: 모든 도구 권한 자동 승인
  - `--sandbox`: 터미널 제한 샌드박스
  - `--add-dir <dir>` (반복 가능): 워크스페이스 디렉터리 추가
  - `--output-format text|json|stream-json`, `--json-schema <schema|path>`: 구조화 출력
  - `--print-timeout <dur>`: 이론상 시간 제한이나 omc 소스는 "non-functional"로 기록 [확인: 소스 주석, 실제 동작은 미검증]
  - `--continue`, `--conversation <id>`, `--project`, `--log-file`
- 작업 디렉터리: `-C` 같은 플래그가 없다. **프로세스 cwd가 워크스페이스**로 보이며 `--add-dir`로 추가 가능. `omc ask`는 `process.cwd()`에서 실행한다. [확인: 플래그 부재, 동작은 미검증]
- 비대화형에서 편집을 허용하려면 `--dangerously-skip-permissions` 또는 `--mode accept-edits`/`auto`가 필요할 것으로 보인다. 조합별 동작은 [미확인].
- 알려진 문제: non-TTY 출력 유실/행(antigravity-cli#76, 1.0.15/1.1.1에서 수정되었다고 omc 소스 주석에 기록). **외부 타임아웃을 건다**.
- 종료 코드: 문서 없음 [미확인]. omc는 "종료 0 + 빈 출력"을 실패로 간주한다.

## 5. 권장 핸드오프 명령 형태 (j-zip, 모델 고정)

전제: 작업 디렉터리 `D:\workspace\test-space\github\j-zip`(git 저장소), 오케스트레이터(Claude Sonnet)가 Bash/PowerShell 도구로 실행. 아래는 **실행해 보지 않은 제안 형태**이며, 첫 사용 시 짧은 스모크 테스트(예: "README 첫 줄을 말해라")로 검증한다. [미확인]

### 5.1 Codex (`gpt-6-luna`) - 코드 작업(편집 허용)
```powershell
$out = ".omc/artifacts/handoff/codex-<task-id>.md"
New-Item -ItemType Directory -Force .omc/artifacts/handoff | Out-Null
codex exec -m gpt-6-luna -c model_reasoning_effort="medium" `
  -s workspace-write -C "D:\workspace\test-space\github\j-zip" `
  -o $out "<.omc/handoffs/<task-id>.md 를 읽고 수행하라. 완료 시 변경 파일/테스트/미해결 항목을 요약하라>"
```
- 읽기 전용 자문/리뷰는 `-s read-only`.
- 긴 프롬프트: `Get-Content task.md -Raw | codex exec -m gpt-6-luna -s workspace-write -o $out -`
- 이벤트 로그가 필요하면 `--json`을 추가하고 stdout을 `.jsonl`로 저장.
- 결과 위치: (a) 최종 메시지 -> `-o` 파일 및 stdout, (b) 코드 변경 -> 작업 트리 직접 수정(`git diff`로 검수).
- 대안 `omc ask codex "<prompt>"`는 모델 고정 불가 + 샌드박스 전부 해제이므로 비권장.

### 5.2 Antigravity (`gemini-3.8-flash-medium`) - 편집 허용
```powershell
agy --model gemini-3.8-flash-medium --mode accept-edits `
  -p "<.omc/handoffs/<task-id>.md 를 읽고 수행하라. 완료 시 변경 파일/테스트/미해결 항목을 요약하라>" `
  --output-format text > .omc/artifacts/handoff/agy-<task-id>.md
```
- 비대화형에서 승인 프롬프트로 멈추면 `--dangerously-skip-permissions`(omc가 쓰는 방식)로 전환하되 범위가 넓으므로 지시문에 수정 허용 경로를 명시한다.
- 외부 타임아웃 필수(Bash 도구 `timeout`, 또는 `run_in_background` + 감시). omc의 선례는 300000ms.
- 결과 위치: 응답은 stdout만(파일 출력 옵션 없음) -> 오케스트레이터가 리다이렉트해 저장. 코드 변경은 작업 트리에 직접 반영.
- 대안 `omc ask antigravity "<prompt>"`는 모델 고정 불가(기본 모델), 대신 아티팩트 자동 저장/5분 타임아웃/빈 출력 검사를 제공한다. 모델 고정이 필수가 아니면 선택지.

### 5.3 공통 권장
- 태스크 명세는 `.omc/handoffs/<task-id>.md`(Decided/Rejected/Risks/Files/Remaining 형식 재사용) 또는 `docs/` 명세 파일 경로로 전달하고, 프롬프트에는 경로와 수용 기준만 적는다.
- 결과 수신 규약 제안: 외부 에이전트에게 "마지막 메시지에 변경 파일 목록, 실행한 테스트, 미해결 항목을 적어라"를 요구하고, 오케스트레이터는 `git status`/`git diff`로 독립 검증한다.
- 병렬 실행 시 같은 파일 동시 수정을 피하도록 파일 소유권을 분리(또는 git worktree). Codex `--worktree`는 관리형 worktree를 새로 만든다(help 문구). [확인]
- 실행 전 `git status`가 깨끗한지 확인하고 실행 후 diff를 검수한다(권한이 넓음).

## 6. 핵심 제한 사항 요약
1. `omc ask`는 모델 지정 불가 -> 지정 모델은 원시 CLI 직접 호출 필요(SKILL 규칙과 상충, 계획서에 예외 명시).
2. `omc ask` 경유 시 외부 에이전트는 샌드박스/승인 우회 상태로 저장소를 수정한다.
3. `omc team`(외부 CLI 워커)은 tmux/psmux 부재로 이 머신에서 사용 불가.
4. agy는 프롬프트를 stdin으로 받지 못하며(argv 길이/인용 주의), 비-TTY 행 이력이 있어 외부 타임아웃이 필요. 파일 출력 옵션 없음.
5. Codex `exec` 기본 샌드박스는 read-only(편집하려면 `-s workspace-write`), `--full-auto` 폐지.
6. 모델 슬러그 가용성(`gpt-6-luna`, `gemini-3.8-flash-medium`)은 카탈로그/`agy models`상 확인됐으나 실제 호출 성공 여부는 미검증.
7. 종료 코드 체계(Codex/agy)는 공식 문서에 없음 -> 0/비0 + 출력 파일 존재 여부로 판단.
8. `.omc/`는 `.gitignore` 대상이므로 영구 보존이 필요한 결과는 `docs/` 등으로 복사.

## 7. 출처
- 로컬: `C:\Users\wnwjdals7498\.claude-company\plugins\cache\omc\oh-my-claudecode\5.7.2\` 하위 `skills\ask\SKILL.md`, `skill-bodies\ask\SKILL.md`, `src\cli\ask.ts`, `scripts\run-provider-advisor.js`, `src\team\model-contract.ts`, `skill-bodies\team\SKILL.md`, `skill-bodies\ultragoal\SKILL.md`, `docs\REFERENCE.md`
- 로컬: `C:\Users\wnwjdals7498\.claude-company\CLAUDE.md`, `C:\Users\wnwjdals7498\.claude-company\.omc-config.json`, `C:\Users\wnwjdals7498\.codex\config.toml`, `C:\Users\wnwjdals7498\.codex\models_cache.json`
- CLI 도움말: `omc --version`, `omc ask --help`, `omc team --help`, `codex --version`, `codex --help`, `codex exec --help`, `agy --version`, `agy --help`, `agy models`
- 공식 문서(Codex 비대화형 모드): https://developers.openai.com/codex/noninteractive (308 리다이렉트 -> https://learn.chatgpt.com/docs/non-interactive-mode 에서 내용 확인)
- Antigravity 설치/문서: https://antigravity.google (SKILL이 참조, 이번 조사에서 직접 열람하지 않음) [미확인]