# G0-INFRA-T01 레포 골격

## 0. 메타

| 항목 | 내용 |
|---|---|
| ID | G0-INFRA-T01 |
| 제목 | 레포 골격: 폴더 구조, 7-Zip 26.04 subtree(`upstream/7zip`, 핸드오프 전 단계에서 오케스트레이터가 가져옴), `upstream/PATCHES.md`, `third_party/VERSIONS.md`, `THIRD_PARTY_NOTICES.md` 초안 |
| 단계 | G0 |
| 크기 | S |
| 상태 | 초안 |
| 권장 구현 모델 | antigravity-gemini3.8flash-medium — 명령 순서와 파일 내용이 이 카드에 전부 고정돼 있어 판단할 일이 적은 정형 작업이다 |
| 검토 모델 | codex-gpt6-luna |
| 선행 작업 | G0-INFRA-T16 (핸드오프 경로 스모크 테스트가 통과해야 외부 모델에 넘길 수 있다, 11 2절) |
| 후행 작업 | 직접: G0-INFRA-T02, G0-INFRA-T08, G0-INFRA-T15 / 간접: G0의 나머지 모든 작업, G0-CLEANUP-T01, G1 이후 전부 |
| 병렬 가능 작업 | G1-SP6-T01, G2-INFRA-T01 (둘 다 11에서 선행 `—`이고 `work/decisions/`에 ADR 파일만 추가한다. T01이 만든 `work/decisions/.gitkeep`을 지워도 충돌로 보지 않는다) |
| 기준 커밋 | —(핸드오프 시 채움) |

## 1. 배경

### 1.1 왜 이 작업이 있나

j-zip의 모든 구현 작업은 [09 2절](../../docs/09-development-process.md)이 정한 폴더 구조와 7-Zip 원본 위치(`upstream/7zip`)를 전제로 한다. 이 작업이 그 바탕을 한 번에 만든다. 이 작업이 병합되기 전에는 빌드(T02), 화면 캡처 도구(T08), 템플릿(T15)을 시작할 수 없다. 외부 구현 모델은 네트워크를 쓸 수 없으므로 7-Zip 원본은 **오케스트레이터가 핸드오프 전에 가져오고**(7.1절 PRE-1·PRE-2), 구현자는 그 위에 폴더와 문서를 만들고 검사한다.

### 1.2 이미 정해진 결정

| 결정 | 근거 |
|---|---|
| 7-Zip 원본은 git subtree로 `upstream/7zip`에 둔다. 26.04부터 시작한다 | [09 2절](../../docs/09-development-process.md), [07 3절](../../docs/07-tech-stack.md) |
| 원본을 고치는 모든 변경은 `// JZIP-PATCH(<작업 ID>): <이유>` … `// JZIP-PATCH-END` 로 감싸고 `upstream/PATCHES.md`에 한 줄로 기록한다. `PATCHES.md`는 subtree prefix 밖(`upstream/PATCHES.md`)에 둔다 | [09 2절](../../docs/09-development-process.md) |
| 동봉 라이브러리는 버전·해시를 `third_party/VERSIONS.md`에 기록하고 `THIRD_PARTY_NOTICES.md`에 고지한다. 갱신은 ADR | [08 7.2절](../../docs/08-architecture.md), [07 7절](../../docs/07-tech-stack.md) |
| j-zip 전체는 7-Zip을 따라 GNU LGPL(unRAR 제한 승계) | [07 7절](../../docs/07-tech-stack.md) "라이선스 원칙" |
| 업스트림 갱신은 `git subtree pull --prefix=upstream/7zip … --squash`로 별도 작업에서 한다 | [09 10절](../../docs/09-development-process.md) |
| subtree 가져오기(upstream, `third_party/`)는 오케스트레이터만 하는 "원본 가져오기"다. 카드에 고정된 원본·태그·해시에서만 가져오고, 해시로 검증하고, `RUNLOG.md`에 기록한다. 외부 구현자는 네트워크를 쓰지 않는다 | [09 2절](../../docs/09-development-process.md), [12 4.10](../../docs/12-agent-orchestration.md) |
| `main`에는 기본적으로 squash 병합한다. **git subtree를 추가·갱신하는 PR(이 카드)은 병합 커밋으로 병합한다**(trailer 커밋이 `main`에 남아야 한다). 직접 push는 금지 | [09 7절](../../docs/09-development-process.md) |

### 1.3 구현에 필요한 사실 (인라인)

| 사실 | 값 | 확인 방법 |
|---|---|---|
| 원본 저장소 | `https://github.com/ip7z/7zip` | [research](../../docs/research/7zip-26.04-source-facts.md) 3줄 |
| 26.04 태그 | `26.04` (가벼운 태그), 커밋 `9128b80e3a471108678e2b3ec3c985861c8d7b0f` | `git ls-remote --tags https://github.com/ip7z/7zip.git`, 2026-10-14 확인 |
| 26.04 루트 트리 해시 | `2d5ed6d8d71737408bc0ddf99837171610ac215b` | `git fetch --depth 1 --filter=blob:none <저장소> refs/tags/26.04` 뒤 `git rev-parse FETCH_HEAD^{tree}` |
| 26.04 파일 수 | 1294개 | `git ls-tree -r --name-only FETCH_HEAD` 줄 수 |
| 루트 구성 | `.gitattributes`, `Asm/`, `C/`, `CPP/`, `DOC/`, `README.md` | `git ls-tree FETCH_HEAD --name-only` |
| 원본 blob의 줄 끝 | LF (`CPP/Build.mak`, `CPP/7zip/Archive/Zip/ZipItem.cpp`에서 CR 0개 확인) | `git cat-file -p FETCH_HEAD:<경로>` |
| 원본 라이선스 | GNU LGPL(2.1 이상). 예외: `CPP/7zip/Compress/Rar*` = LGPL + unRAR 제한, `CPP/7zip/Compress/LzfseDecoder.cpp`·`C/ZstdDec.c` = BSD 3-clause, `C/Xxh64.c` = BSD 2-clause | `upstream/7zip/DOC/License.txt` |
| 이 환경의 실험 결과 | 임시 레포에서 `git subtree add --prefix=upstream/7zip https://github.com/ip7z/7zip.git 26.04 --squash -m "<메시지>"`를 실행하면 약 8초에 끝나고, `git rev-parse HEAD:upstream/7zip`이 위 트리 해시와 같고, 병합 커밋 아래에 `git-subtree-dir: upstream/7zip`·`git-subtree-split: 9128b80e…` trailer를 가진 "Squashed" 커밋이 생긴다. 병합 커밋의 첫째 부모는 가져오기 직전의 `HEAD`다 | 카드 작성자 확인 |
| 7-Zip 원본 경로 인용 | 이후 모든 카드가 `upstream/7zip/CPP/7zip/Archive/Zip/ZipItem.cpp:405`처럼 인용한다. 이 줄 번호가 맞아야 카드가 맞다 | 6.7절 TC-04 |

## 2. Goal

| ID | 목표 | 검증 |
|---|---|---|
| GL-1 | [09 2절](../../docs/09-development-process.md) 구조의 폴더 20개(8절 TC-01 목록, `spikes/`와 `tests/scratch/` 포함)가 있고, 빈 폴더는 `.gitkeep` 파일 하나를 가진다 | TC-01, DC-3 |
| GL-2 | `upstream/7zip`의 내용이 7-Zip 26.04 태그와 한 바이트도 다르지 않다(오케스트레이터가 PRE-1·PRE-2로 가져와 검증하고, 구현자의 검사 스크립트가 다시 확인한다): 루트 트리 해시 `2d5ed6d8d71737408bc0ddf99837171610ac215b`, 추적 파일 1294개, 작업 트리 변경 0 | TC-03, TC-04 |
| GL-3 | 브랜치 이력에 `git-subtree-dir: upstream/7zip`와 `git-subtree-split: 9128b80e3a471108678e2b3ec3c985861c8d7b0f` trailer를 가진 커밋이 있고, 병합 커밋으로 병합한 뒤 `main` 이력에도 남는다 | TC-06, DC-7 |
| GL-4 | `upstream/PATCHES.md`가 6.3절 형식으로 있다(기준 표 + 빈 패치 표) | TC-02, TC-05 |
| GL-5 | `third_party/VERSIONS.md`가 6.5절 내용으로 있다 | TC-02, TC-05 |
| GL-6 | `THIRD_PARTY_NOTICES.md` 초안이 6.6절 내용으로 있고, 7-Zip 라이선스 구조가 `upstream/7zip/DOC/License.txt`와 일치한다 | TC-02, TC-05, DC-6 |
| GL-7 | `.gitignore`에 `out/` 한 줄이 있다 | TC-02 |
| GL-8 | 기준 커밋 이후 바뀐 경로가 전부 5.1절 허용 경로 안에 있다 | TC-07 |
| GL-9 | 검사 스크립트가 구현자의 변경 전에는 실패(종료 코드 1)하고 변경 후에는 통과(종료 코드 0)한 로그와 `EVIDENCE.md`가 있다 | 10절 E1' |

## 3. Non-Goal

| ID | 제외 내용 | 소유 작업 |
|---|---|---|
| NG-1 | `tools/build.ps1`, `jzip.sln`, 빌드 구성 4종(Debug·Dev·Test·Release), nmake 래핑, `out/runlog.jsonl` | G0-INFRA-T02 |
| NG-2 | 기준선 빌드(원본과 같은 `7z.exe`·`7z.dll` 빌드와 코퍼스 v0 통과, [09 3.2](../../docs/09-development-process.md) #1). 이 카드는 원본을 빌드하지 않는다 | G0-INFRA-T02(빌드), G0-INFRA-T03(CI), G0-INFRA-T05(코퍼스) |
| NG-3 | `.github/` 아래 CI 워크플로 | G0-INFRA-T03 |
| NG-4 | GoogleTest, `tests/unit/run.ps1` | G0-INFRA-T04 |
| NG-5 | 코퍼스 생성기 `tests/corpus/generate.ps1`, 통합 테스트 실행기 `tests/integration/run.ps1` | G0-INFRA-T05 |
| NG-6 | 로그 뼈대(`ILogger`, `JZ_LOG_*`, ETW), `src/core/infra/log/EVENTS.md`, `tools/evidence/trace-start.ps1`·`trace-stop.ps1` | G0-INFRA-T06 |
| NG-7 | `tools/evidence/new-manifest.ps1`, `JZIP-VERIFY` 반영 봇 | G0-INFRA-T07 |
| NG-8 | `tools/evidence/capture-window.ps1` | G0-INFRA-T08 |
| NG-9 | `tests/perf/run.ps1` | G0-INFRA-T09 |
| NG-10 | `tools/gate/check-task.ps1`, `check-stage.ps1` | G0-INFRA-T10 |
| NG-11 | `jzip-shared`·`jzip-win` 빈 라이브러리, nmake 변수 `JZIP_SHARED_LIB`·`JZIP_WIN_LIB` | G0-INFRA-T11 |
| NG-12 | `jzip-shell.dll` 뼈대와 링크 검사 | G0-INFRA-T12 |
| NG-13 | WinUI 3 앱 `JZip.exe`, MSIX 뼈대 | G0-INFRA-T14 |
| NG-14 | PR 템플릿, ADR 템플릿, 카드 템플릿 파일, `JZIP-VERIFY` 댓글 형식 파일 | G0-INFRA-T15 (카드 표준 문서는 이미 `work/tasks/README.md`에 있다) |
| NG-15 | `third_party/lz4/`, `third_party/unalz/` 소스 동봉 | G5-F08-T01(LZ4), G4-F07-T01(unalz) |
| NG-16 | `upstream/7zip/` 안 파일의 수정(`JZIP-PATCH`). 이 카드에서 원본 변경은 0줄이다 | 각 기능 작업(예: G2-F17-T04) |
| NG-17 | `THIRD_PARTY_NOTICES.md` 최종본, LGPL 소스 공개 링크 | G6-INFRA-T03 |
| NG-18 | `spikes/`·`tests/scratch/` 안에 실제 파일을 만드는 일과 단계 종료 때의 정리(`.gitkeep` 외 파일 삭제). 이 카드는 두 폴더에 `.gitkeep`만 둔다 | 처음 그 폴더를 쓰는 작업(G1 시제품 작업), 단계 정리 G1-CLEANUP-T01 |
| NG-19 | 줄 끝 정책 파일 `.gitattributes` | 소유 작업 없음. 09·10에 정책이 없다. 필요해지면 ADR로 정한다 |
| NG-20 | `git subtree add` 실행, 출력 로그 저장, 해시·trailer 검증, RUNLOG 기록. 구현자는 이 일을 하지 않고 네트워크를 쓰지 않는다 | 오케스트레이터(7.1절 PRE-1·PRE-2, [12 4.10](../../docs/12-agent-orchestration.md)) |

## 4. 근거

| 종류 | 근거 |
|---|---|
| 수용 기준 | [09 3.2](../../docs/09-development-process.md) **#12 레포 골격**(2절 구조의 폴더, `upstream/7zip`이 26.04 태그와 트리 해시까지 같음, `PATCHES.md`·`VERSIONS.md`·`THIRD_PARTY_NOTICES.md` 존재). [09 4절](../../docs/09-development-process.md) 표가 `G0-INFRA` 카드에 요구하는 게이트 항목 번호다 |
| 구조 | [09 2절](../../docs/09-development-process.md) 레포 구조와 원본 수정 규칙 |
| 절차 | [09 2절](../../docs/09-development-process.md) 원본 가져오기, [09 7절](../../docs/09-development-process.md) 브랜치·커밋·PR·병합 방식, [09 10절](../../docs/09-development-process.md) 업스트림 갱신, [12 4.10](../../docs/12-agent-orchestration.md) 원본 가져오기 |
| 기록 형식 | [10 4절](../../docs/10-logging-evidence-operations.md) `JZIP-PATCH`가 `upstream/PATCHES.md`에 기록됐는지 검사, [10 6절](../../docs/10-logging-evidence-operations.md) 출시 체크리스트의 `THIRD_PARTY_NOTICES.md`·`PATCHES.md` |
| 의존성 | [08 7.2절](../../docs/08-architecture.md) 공급망 행, [07 7절](../../docs/07-tech-stack.md) 라이선스 원칙 |
| 증거 | [10 3.2절](../../docs/10-logging-evidence-operations.md) 필수 증거와 "빌드할 코드가 없는 작업"의 E1 대체 규칙, [10 3.3절](../../docs/10-logging-evidence-operations.md) 수동 작성 매니페스트, [11 2절](../../docs/11-implementation-backlog.md) G0-INFRA-T01 행(증거 E1) |
| 원본 | `upstream/7zip/DOC/License.txt`, `upstream/7zip/DOC/unRarLicense.txt`, 커밋 `9128b80`([research](../../docs/research/7zip-26.04-source-facts.md)) |
| ADR | 없음 |

## 5. 변경 범위

### 5.1 허용 경로

| 경로 | 허용 작업 |
|---|---|
| `upstream/7zip/**` | 오케스트레이터의 PRE-1 `git subtree add`가 만든 추가만. 구현자는 어떤 수정도 하지 않는다 |
| `upstream/PATCHES.md` | 새로 만듦 |
| `third_party/VERSIONS.md` | 새로 만듦 |
| `THIRD_PARTY_NOTICES.md` | 새로 만듦 |
| `.gitignore` | `out/` 한 줄 추가(이미 있는 줄은 그대로 둔다) |
| `src/shared/.gitkeep`, `src/shared-win/.gitkeep`, `src/core/.gitkeep`, `src/app/.gitkeep`, `src/shell/.gitkeep`, `src/engine/.gitkeep` | 새로 만듦 |
| `tests/unit/.gitkeep`, `tests/integration/.gitkeep`, `tests/perf/.gitkeep`, `tests/corpus/.gitkeep`, `tests/scratch/.gitkeep` | 새로 만듦 |
| `spikes/.gitkeep` | 새로 만듦 |
| `tools/gate/.gitkeep`, `tools/evidence/.gitkeep` | 새로 만듦 |
| `work/decisions/.gitkeep`, `work/evidence/.gitkeep`, `packaging/.gitkeep` | 새로 만듦 |
| `work/evidence/G0-INFRA-T01/**` | 증거 파일(별도 커밋, 10절) |
| `work/handoff/G0-INFRA-T01/RESULT.md` | 구현 결과 보고(R-1에서 만들고 갱신, [12 4.3](../../docs/12-agent-orchestration.md) 항상 허용) |

### 5.2 허용 upstream 파일

없음 — `upstream/7zip/**`의 어떤 파일도 수정하지 않는다. `JZIP-PATCH`는 0건이다.

### 5.3 금지 경로

위 5.1에 없는 모든 경로. 특히 `docs/**`, `design/**`, `work/tasks/**`, `work/handoff/G0-INFRA-T01/PACKET.md`, `work/handoff/G0-INFRA-T01/RUNLOG.md`·`import-*.log`(오케스트레이터의 기록), `.github/**`, `tools/*.ps1`, `jzip.sln`, `README.md`.

### 5.4 새로 만드는 파일

| 경로 | 역할 | 직접 쓰는 줄 수 |
|---|---|---|
| `upstream/PATCHES.md` | 원본 패치 대장(6.3절) | 약 30 |
| `third_party/VERSIONS.md` | 동봉 라이브러리 버전 대장(6.5절) | 약 25 |
| `THIRD_PARTY_NOTICES.md` | 서드파티 고지 초안(6.6절) | 약 60 |
| `.gitkeep` 17개 | 빈 폴더 유지(5.1절 표 경로) | 0 |
| `work/evidence/G0-INFRA-T01/verify-skeleton.ps1` | 검사 스크립트(6.7절) | 104 |
| `work/evidence/G0-INFRA-T01/verify-before.log`, `verify-after.log` | 실행 로그 | — |
| `work/evidence/G0-INFRA-T01/EVIDENCE.md` | 매니페스트(10절) | 약 45 |

## 6. 인터페이스 계약

### 6.1 폴더 계약 — 확정(09 2절 레포 구조)

| 폴더 | 09 2절의 용도 | 초기 내용 |
|---|---|---|
| `upstream/7zip/` | 7-Zip 원본(git subtree prefix) | subtree 내용 |
| `src/shared/` | `jzip-shared.lib` 순수 도메인 로직 | `.gitkeep` |
| `src/shared-win/` | `jzip-win.lib` Win32 포트 구현 | `.gitkeep` |
| `src/core/` | `jzip-core.lib` 응용·기반 | `.gitkeep` |
| `src/app/` | `JZip.exe` WinUI 3 화면 | `.gitkeep` |
| `src/shell/` | `jzip-shell.dll` | `.gitkeep` |
| `src/engine/` | `7z.dll`·`7z.exe` 빌드 래퍼 + 추가 핸들러 | `.gitkeep` |
| `third_party/` | lz4, unalz | `VERSIONS.md` (`.gitkeep` 없음) |
| `tests/unit/`, `tests/integration/`, `tests/perf/`, `tests/corpus/` | 영구 테스트와 코퍼스 | `.gitkeep` |
| `tests/scratch/` | 탐색용 테스트(단계 종료 때 정리) | `.gitkeep` |
| `spikes/` | 시제품 코드(단계 종료 때 삭제) | `.gitkeep` |
| `tools/gate/`, `tools/evidence/` | 검사·증거 스크립트 | `.gitkeep` |
| `work/tasks/` | 작업 카드 | 이미 있음 (이 카드의 변경 아님) |
| `work/decisions/`, `work/evidence/` | ADR, 증거 매니페스트 | `.gitkeep` |
| `packaging/` | MSIX 매니페스트, 서명 설정 | `.gitkeep` |

`.gitkeep` 사용은 **제안(카드 결정)** 이다. 이유: git은 빈 폴더를 추적하지 못한다. `spikes/`와 `tests/scratch/`에도 `.gitkeep`을 둔다. [10 5절](../../docs/10-logging-evidence-operations.md)의 "비어 있음"은 `.gitkeep` 외 파일이 없다는 뜻이라 단계 종료 검사와 충돌하지 않는다. 나중에 폴더에 실제 파일을 넣는 작업은 같은 PR에서 그 폴더의 `.gitkeep`을 지울 수 있다. 단 `spikes/`·`tests/scratch/`의 `.gitkeep`은 지우지 않는다(정리 뒤에도 폴더가 남아야 한다).

### 6.2 subtree 계약

확정(09 2·7·10절, 12 4.10, 7-Zip 사실): 위치 `upstream/7zip`, 7-Zip 26.04, 커밋 메시지에 작업 ID 포함. **실행 주체는 오케스트레이터**(7.1절 PRE-1)이고 구현자는 이 명령을 실행하지 않는다.
제안(카드 결정): `--squash` 사용. 이유: 7-Zip 전체 이력을 j-zip 저장소에 들이지 않기 위해서다. 한 번 정하면 이후 `pull`도 같은 플래그를 써야 하므로 [09 10절](../../docs/09-development-process.md)이 `--squash`를 명시하고 `upstream/PATCHES.md` 기록 규칙(6.3절)에도 명령을 적는다. 이 PR은 squash하지 않고 병합 커밋으로 병합한다([09 7절](../../docs/09-development-process.md)). 그래야 아래 trailer 커밋이 `main` 이력에 남는다.

```
git subtree add --prefix=upstream/7zip https://github.com/ip7z/7zip.git 26.04 --squash -m "chore(upstream): import 7-Zip 26.04 [G0-INFRA-T01]"
```

- Input: 깨끗한 작업 트리(추적 파일 변경 없음. 추적하지 않는 파일은 있어도 된다), 브랜치 `chore/G0-INFRA-T01-repo-skeleton`, PACKET 커밋이 이미 있는 상태. 고정된 원본은 위 명령의 저장소·태그(`26.04`)와 1.3절의 커밋·트리 해시다.
- Output: 병합 커밋 1개(위 메시지)와 그 아래 "Squashed 'upstream/7zip/' content from commit 9128b80" 커밋 1개. 아래 커밋 본문에 trailer가 있다.

```
git-subtree-dir: upstream/7zip
git-subtree-split: 9128b80e3a471108678e2b3ec3c985861c8d7b0f
```

- 오류: 네트워크 실패, `git subtree` 명령 없음, 작업 트리 변경 있음 → 명령이 0이 아닌 코드로 끝난다. 오케스트레이터가 원인을 해결하고 PRE-1을 다시 한다(7.1절). 구현자는 구현을 시작하지 않는다.
- 부작용: `upstream/7zip/` 1294개 파일 추가.

### 6.3 `upstream/PATCHES.md` 형식 — 제안(열 구성은 카드 결정, "한 줄로 기록"은 09 2절 확정)

파일 전체 내용(그대로 만든다):

```markdown
# upstream/7zip 패치 목록

`upstream/7zip`(7-Zip 원본)에 들어간 j-zip 변경의 대장이다. 원본 수정 규칙은 [09. 구현 절차](../docs/09-development-process.md) 2절이다.

## 기준

| 항목 | 값 |
|---|---|
| 원본 저장소 | https://github.com/ip7z/7zip |
| 버전 | 26.04 (`C/7zVersion.h`의 `MY_VERSION_NUMBERS "26.04"`) |
| 태그 | `26.04` |
| 커밋 | `9128b80e3a471108678e2b3ec3c985861c8d7b0f` |
| 루트 트리 | `2d5ed6d8d71737408bc0ddf99837171610ac215b` |
| 가져온 명령 | `git subtree add --prefix=upstream/7zip https://github.com/ip7z/7zip.git 26.04 --squash` |
| 가져온 작업 | G0-INFRA-T01 (오케스트레이터가 핸드오프 전에 가져옴) |

## 기록 규칙

- 원본 파일을 고칠 때마다 바뀐 부분을 `// JZIP-PATCH(<작업 ID>): <이유>` 와 `// JZIP-PATCH-END` 로 감싸고, 아래 표에 `JZIP-PATCH` 블록 하나당 한 줄을 추가한다.
- `작업 ID`: 블록의 `JZIP-PATCH(<작업 ID>)` 값과 같다.
- `파일`: 레포 루트 기준 경로(`upstream/7zip/...`).
- `위치`: 블록이 들어 있는 함수 또는 클래스 이름. 줄 번호는 업스트림 갱신 때 바뀌므로 쓰지 않는다.
- `이유`: 블록의 `<이유>`와 같은 문장.
- 작업 카드의 "허용 upstream 파일"에 없는 파일은 고치지 않는다.
- 업스트림을 갱신할 때는 오케스트레이터가 `git subtree pull --prefix=upstream/7zip https://github.com/ip7z/7zip.git <태그> --squash` 를 별도 작업(`G<n>-UPSTREAM-T<번호>`)으로 실행하고([09](../docs/09-development-process.md) 10절, [12](../docs/12-agent-orchestration.md) 4.10), 위 "기준" 표를 새 값으로 바꾼다. 이 작업의 PR은 병합 커밋으로 병합한다([09](../docs/09-development-process.md) 7절).

## 패치 목록

| 작업 ID | 파일 | 위치 | 이유 |
|---|---|---|---|
```

- 표의 머리글 줄은 정확히 `| 작업 ID | 파일 | 위치 | 이유 |`, 다음 줄은 정확히 `|---|---|---|---|` 이다. `G0-INFRA-T10`의 `check-task`가 이 두 줄을 찾아 행을 읽는다. 행이 0개인 것이 정상이다.

### 6.4 `JZIP-PATCH` 표식 형식 — 확정(09 2절, 10 4절)

```
// JZIP-PATCH(<작업 ID>): <이유>
… 바꾼 코드 …
// JZIP-PATCH-END
```

`<작업 ID>`는 `^G[0-6]-(F\d{2}|SP\d|INFRA|OPS|CLEANUP|UPSTREAM)-T\d{2}$`를 따른다. 이 카드는 표식을 코드에 넣지 않는다. 형식만 6.3절 규칙에 옮겨 적는다.

### 6.5 `third_party/VERSIONS.md` — 제안(카드 결정, 기록 의무는 08 7.2절 확정)

파일 전체 내용(그대로 만든다):

```markdown
# third_party 버전 대장

`third_party/`에 소스로 동봉하는 라이브러리의 버전과 출처를 기록한다. 동봉하는 작업이 이 표의 행을 채운다. 갱신은 ADR로 한다([08](../docs/08-architecture.md) 7.2절).

## 소스로 동봉하는 라이브러리

| 이름 | 버전 | 상태 | 출처 | 태그·커밋 | 아카이브 SHA-256 | 라이선스 | 용도 | 동봉 작업 |
|---|---|---|---|---|---|---|---|---|
| LZ4 | 1.10.0 | 예정 | 동봉 작업에서 기록 | 동봉 작업에서 기록 | 동봉 작업에서 기록 | BSD-2-Clause | #8 LZ4 해제 | G5-F08-T01 |
| unalz | 동봉 작업에서 정함 | 예정 | 동봉 작업에서 기록 | 동봉 작업에서 기록 | 동봉 작업에서 기록 | zlib | #7 ALZ 해제 | G4-F07-T01 |

- `상태`: `예정`(아직 `third_party/`에 없음) / `동봉`(소스가 들어 있고 위 칸이 모두 채워짐).
- `동봉` 행은 `third_party/<이름>/LICENSE` 파일이 있어야 한다([09](../docs/09-development-process.md) 2절).
- 소스를 `third_party/`에 들이는 일은 오케스트레이터만 하는 원본 가져오기다([12](../docs/12-agent-orchestration.md) 4.10). `출처`, `태그·커밋`, `아카이브 SHA-256`은 가져올 때 고정한 값이며, 카드에 적힌 값과 같아야 한다.

## 소스로 동봉하지 않는 의존성 (기록 위치만 안내)

| 의존성 | 고정 위치 | 만드는 작업 |
|---|---|---|
| GoogleTest | vcpkg 매니페스트의 `builtin-baseline` | G0-INFRA-T04 |
| Microsoft.WindowsAppSDK, Microsoft.Windows.CppWinRT, Microsoft.Windows.ImplementationLibrary | NuGet 잠금 파일 | G0-INFRA-T14 |
```

값의 출처: LZ4 1.10.0·BSD-2-Clause·#8, unalz·zlib·#7은 [07 7절](../../docs/07-tech-stack.md) "서드파티" 표. 동봉 작업 ID는 [11](../../docs/11-implementation-backlog.md) G5-F08-T01·G4-F07-T01. 고정 위치는 [08 7.2절](../../docs/08-architecture.md) 공급망 행.

### 6.6 `THIRD_PARTY_NOTICES.md` 초안 — 제안(구조는 카드 결정, 라이선스 사실은 `DOC/License.txt` 확정)

파일 전체 내용(그대로 만든다):

```markdown
# 서드파티 고지 (초안)

이 문서는 j-zip이 포함하거나 링크하는 외부 소프트웨어의 저작권과 라이선스를 알린다. 지금은 초안이며, 배포 구성이 정해지면 G6-INFRA-T03에서 최종본으로 바꾼다. 근거: [07](docs/07-tech-stack.md) 7절 "라이선스 원칙", [10](docs/10-logging-evidence-operations.md) 6절.

j-zip은 7-Zip을 따라 GNU LGPL로 배포한다. 7-Zip의 unRAR 제한도 그대로 이어받는다.

## 1. 7-Zip

| 항목 | 내용 |
|---|---|
| 저작권 | 7-Zip Copyright (C) 1999-2026 Igor Pavlov |
| 위치 | `upstream/7zip` |
| 버전 | 26.04 (커밋 `9128b80e3a471108678e2b3ec3c985861c8d7b0f`) |
| 기본 라이선스 | GNU LGPL 2.1 이상 — `upstream/7zip/DOC/License.txt`, `upstream/7zip/DOC/copying.txt` |
| j-zip 변경 | `upstream/PATCHES.md`에 기록 |

파일별 예외(`upstream/7zip/DOC/License.txt` 기준):

| 대상 | 라이선스 | 저작권자 |
|---|---|---|
| `CPP/7zip/Compress/Rar*` | GNU LGPL + unRAR 라이선스 제한(`upstream/7zip/DOC/unRarLicense.txt`) | Igor Pavlov, Alexander Roshal(unRAR) |
| `CPP/7zip/Compress/LzfseDecoder.cpp` | BSD 3-clause | Apple Inc., Igor Pavlov |
| `C/ZstdDec.c` | BSD 3-clause | Facebook, Inc., Igor Pavlov |
| `C/Xxh64.c` | BSD 2-clause | Yann Collet, Igor Pavlov |
| 소스 파일에 "public domain"이라고 적힌 파일 | 퍼블릭 도메인 | 해당 파일 참조 |
| 그 밖의 모든 파일 | GNU LGPL | Igor Pavlov |

unRAR 제한: unRAR 소스는 RAR(WinRAR) 호환 압축기를 만드는 데 쓸 수 없다. 수정한 unRAR 소스를 따로 배포하거나 다른 소프트웨어의 일부로 배포할 때는 문서와 소스 주석에 이 제한을 밝혀야 한다.

## 2. 소스로 동봉할 라이브러리 (예정)

| 이름 | 라이선스 | 동봉 작업 | 저작권 문구 |
|---|---|---|---|
| LZ4 1.10.0 | BSD-2-Clause | G5-F08-T01 | 동봉 작업이 `third_party/lz4/LICENSE`에서 옮긴다 |
| unalz | zlib | G4-F07-T01 | 동봉 작업이 `third_party/unalz/LICENSE`에서 옮긴다 |

## 3. Microsoft 구성 요소 (NuGet, 예정)

| 패키지 | 라이선스 |
|---|---|
| Microsoft.WindowsAppSDK | MIT |
| Microsoft.Windows.CppWinRT | MIT |
| Microsoft.Windows.ImplementationLibrary (WIL) | MIT |

## 4. 배포하지 않는 개발 도구

| 이름 | 라이선스 | 용도 |
|---|---|---|
| GoogleTest | BSD-3-Clause | 단위 테스트(배포물 미포함) |

## 5. 소스 공개

LGPL 소스 공개 링크는 최종본(G6-INFRA-T03)에 적는다. j-zip 저장소: https://github.com/wnwjdals7498/j-zip
```

값의 출처: 7-Zip 항목은 `upstream/7zip/DOC/License.txt`와 `unRarLicense.txt`(이 파일에 "All copyrights to RAR and the utility unRAR are exclusively owned by the author - Alexander Roshal"). Microsoft 패키지 라이선스·GoogleTest 라이선스는 [07 7절](../../docs/07-tech-stack.md). 저장소 주소는 `git remote get-url origin`.

### 6.7 검사 스크립트 — 제안(카드 결정)

경로 `work/evidence/G0-INFRA-T01/verify-skeleton.ps1`. 이 작업 전용 증거 파일이며 영구 도구가 아니다(영구 게이트 스크립트는 G0-INFRA-T10).

- 호출: `powershell -NoProfile -File work/evidence/G0-INFRA-T01/verify-skeleton.ps1 -Base <기준 커밋 SHA> [-Repo <경로>]` (`-Repo` 기본값은 현재 폴더)
- 출력: 검사마다 한 줄 `PASS TC-nn <설명>` 또는 `FAIL TC-nn <설명>`, 마지막 줄 `ALL PASS` 또는 `FAILED <실패 수>`
- 종료 코드: 0 = 전부 통과, 1 = 검사 실패, 2 = 사용법 오류([09 11절](../../docs/09-development-process.md) 종료 코드 규칙)
- 스레드·성능: 단일 스레드, 10초 안에 끝난다.
- **파일은 ASCII만 쓴다.** Windows PowerShell 5.1이 BOM 없는 UTF-8을 ANSI로 읽기 때문이다.
- 함수 이름을 `Git`으로 짓지 않는다. 함수가 `git` 명령을 가리면서 자기 자신을 무한 호출한다. 아래 스크립트는 `RunGit` 안에서 `git.exe`를 쓴다.

파일 전체 내용(그대로 만든다):

```powershell
# work/evidence/G0-INFRA-T01/verify-skeleton.ps1
# Checks for task G0-INFRA-T01. Exit code: 0 = all pass, 1 = check failed, 2 = usage error.
# ASCII only on purpose: Windows PowerShell 5.1 reads BOM-less UTF-8 files as ANSI.
param(
  [string]$Repo = (Get-Location).Path,
  [Parameter(Mandatory = $true)][string]$Base
)
Set-StrictMode -Version 2
$ErrorActionPreference = 'Stop'

$TagCommit = '9128b80e3a471108678e2b3ec3c985861c8d7b0f'
$TreeSha   = '2d5ed6d8d71737408bc0ddf99837171610ac215b'
$FileCount = 1294
$script:Failures = 0

function RunGit { & git.exe -C $Repo @args }
function Check([string]$Id, [string]$Text, [scriptblock]$Test) {
  $ok = $false
  try { $ok = [bool](& $Test) } catch { $ok = $false }
  if ($ok) { Write-Output ("PASS {0} {1}" -f $Id, $Text) }
  else     { Write-Output ("FAIL {0} {1}" -f $Id, $Text); $script:Failures++ }
}

if (-not (Test-Path -LiteralPath (Join-Path $Repo '.git'))) {
  [Console]::Error.WriteLine("not a git work tree: $Repo"); exit 2
}
if ($null -eq (RunGit rev-parse --verify --quiet "$Base^{commit}")) {
  [Console]::Error.WriteLine("base commit not found: $Base"); exit 2
}

# TC-01 directories
$dirs = @('src/shared','src/shared-win','src/core','src/app','src/shell','src/engine',
          'third_party','tests/unit','tests/integration','tests/perf','tests/corpus','tests/scratch',
          'spikes','tools/gate','tools/evidence','work/tasks','work/decisions','work/evidence',
          'packaging','upstream/7zip')
foreach ($d in $dirs) {
  Check 'TC-01' "directory $d" { Test-Path -LiteralPath (Join-Path $Repo $d) -PathType Container }
}
# TC-02 new files
foreach ($f in @('upstream/PATCHES.md','third_party/VERSIONS.md','THIRD_PARTY_NOTICES.md')) {
  Check 'TC-02' "file $f" { Test-Path -LiteralPath (Join-Path $Repo $f) -PathType Leaf }
}
Check 'TC-02' '.gitignore has a line: out/' {
  (Get-Content -LiteralPath (Join-Path $Repo '.gitignore')) -contains 'out/'
}
# TC-03 subtree identity
Check 'TC-03' "HEAD:upstream/7zip tree == $TreeSha" { (RunGit rev-parse 'HEAD:upstream/7zip') -eq $TreeSha }
Check 'TC-03' "tracked files under upstream/7zip == $FileCount" {
  @(RunGit ls-files 'upstream/7zip').Count -eq $FileCount
}
Check 'TC-03' 'work tree under upstream/7zip unchanged' { @(RunGit status --porcelain -- 'upstream/7zip').Count -eq 0 }
Check 'TC-03' '7zVersion.h says 26.04' {
  $null -ne (Select-String -LiteralPath (Join-Path $Repo 'upstream/7zip/C/7zVersion.h') `
             -SimpleMatch 'MY_VERSION_NUMBERS "26.04"')
}
# TC-04 line anchors cited by docs F17 section 2, 07 section 5 and card G2-F17-T03
function LineStarts([string]$Rel, [int]$No, [string]$Text) {
  $l = (Get-Content -LiteralPath (Join-Path $Repo $Rel) -TotalCount $No)[$No - 1]
  return $l.StartsWith($Text)
}
Check 'TC-04' 'ZipItem.cpp:405 GetUnicodeString' {
  LineStarts 'upstream/7zip/CPP/7zip/Archive/Zip/ZipItem.cpp' 405 'void CItem::GetUnicodeString('
}
Check 'TC-04' 'UTFConvert.cpp:230 Check_UTF8_Buf' {
  LineStarts 'upstream/7zip/CPP/Common/UTFConvert.cpp' 230 'bool Check_UTF8_Buf('
}
Check 'TC-04' 'Build.mak:80 contains -GR-' {
  (Get-Content -LiteralPath (Join-Path $Repo 'upstream/7zip/CPP/Build.mak') -TotalCount 80)[79].Contains('-GR-')
}
# TC-05 document format
Check 'TC-05' 'PATCHES.md has a 4-column table header and separator' {
  $lines = @(Get-Content -LiteralPath (Join-Path $Repo 'upstream/PATCHES.md'))
  $hit = $false
  for ($i = 0; $i -lt $lines.Count - 1; $i++) {
    if ($lines[$i] -match '^\| [^|]+ \| [^|]+ \| [^|]+ \| [^|]+ \|$' -and $lines[$i + 1] -eq '|---|---|---|---|') { $hit = $true }
  }
  $hit
}
foreach ($f in @('upstream/PATCHES.md','third_party/VERSIONS.md','THIRD_PARTY_NOTICES.md')) {
  Check 'TC-05' "$f has no UTF-8 BOM" {
    $b = [System.IO.File]::ReadAllBytes((Join-Path $Repo $f))
    -not ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF)
  }
  Check 'TC-05' "$f is stored with LF line ends (index eol = lf)" {
    (RunGit ls-files --eol -- $f) -match '^i/lf\s'
  }
}
# TC-06 subtree metadata (the PR is merged with a merge commit, so these commits stay on main)
Check 'TC-06' 'commit with trailer git-subtree-dir: upstream/7zip' {
  @(RunGit log --grep '^git-subtree-dir: upstream/7zip$' --format=%H).Count -ge 1
}
Check 'TC-06' "commit with trailer git-subtree-split: $TagCommit" {
  @(RunGit log --grep "^git-subtree-split: $TagCommit$" --format=%H).Count -ge 1
}
# TC-07 change scope: every path changed since the base commit is on the allowed list
$allowed = '^(upstream/|src/|tests/|tools/|work/|packaging/|spikes/|third_party/|THIRD_PARTY_NOTICES\.md$|\.gitignore$)'
Check 'TC-07' 'no change outside the allowed paths' {
  @(RunGit diff --name-only "$Base..HEAD" | Where-Object { $_ -notmatch $allowed }).Count -eq 0
}
if ($script:Failures -gt 0) { Write-Output ("FAILED {0}" -f $script:Failures); exit 1 }
Write-Output 'ALL PASS'
exit 0
```

카드 작성자가 이 스크립트를 임시 레포(1.3절 실험 레포: 기준 커밋, PACKET 커밋, PRE-1의 `git subtree add`, 기록 커밋 순서)에서 실행해, R-4 시점에는 `FAIL` 28줄과 `FAILED 28`(종료 코드 1: `TC-01` 17줄, `TC-02` 4줄, `TC-05` 7줄)을, R-5·R-6 뒤에는 `PASS` 41줄과 `ALL PASS`(종료 코드 0)를, 레포가 아닌 폴더에서 종료 코드 2를 확인했다(2026-10-14, r1 개정 뒤 다시 실행).

## 7. 동작 명세

작업은 두 부분이다. 먼저 **오케스트레이터가 핸드오프 전 단계(PRE-1·PRE-2)** 로 원본을 가져오고, 그다음 구현자가 R-1~R-9를 순서대로 한다. 순서를 바꾸지 않는다. `git subtree add`는 추적 파일이 깨끗해야 하므로 다른 커밋보다 먼저 한다. 외부 구현 모델은 네트워크를 쓰지 않으므로 원본을 직접 내려받지 않는다([12 4.10](../../docs/12-agent-orchestration.md)).

### 7.1 핸드오프 전 단계 (오케스트레이터 전용)

| 단계 | 규칙 | 출처 |
|---|---|---|
| PRE-1 | worktree를 만들고 PACKET 커밋(`docs(handoff): packet [G0-INFRA-T01]`)을 만든 뒤, `git -C <WT> status --porcelain`이 비어 있음을 확인하고 6.2절 명령을 worktree에서 실행한다. 명령 앞에 `env -u GH_TOKEN -u GITHUB_TOKEN`을 붙이고 출력을 `work/handoff/G0-INFRA-T01/import-1.log`로 저장한다. 네트워크 오류면 한 번 더 실행하고, 두 번째도 실패하면 구현을 시작하지 않고 원인을 해결한다 | [12 4.10](../../docs/12-agent-orchestration.md), [09 2절](../../docs/09-development-process.md) "원본 가져오기" |
| PRE-2 | 검증: ① `git rev-parse HEAD:upstream/7zip`이 `2d5ed6d8d71737408bc0ddf99837171610ac215b`와 같다 ② `git log --grep '^git-subtree-split: 9128b80e3a471108678e2b3ec3c985861c8d7b0f$' --format=%H`가 1개 이상 ③ `git ls-files upstream/7zip`이 1294줄 ④ `git status --porcelain --untracked-files=no`가 비어 있다(`import-1.log`는 기록 커밋 전이라 추적하지 않는 파일로 남는다). 하나라도 다르면 `git -C <WT> reset --hard <PACKET 커밋 SHA>`로 되돌리고 구현을 시작하지 않는다(카드 값이 틀렸는지 원본이 바뀌었는지 먼저 확인한다). 통과하면 `RUNLOG.md`에 역할 `원본 가져오기` 항목([12 4.8](../../docs/12-agent-orchestration.md))을 쓰고, `RUNLOG.md`와 `import-1.log`만 담은 기록 전용 커밋 `docs(handoff): record upstream import [G0-INFRA-T01]`을 만든다 | [12 4.8·4.10](../../docs/12-agent-orchestration.md) |

PRE 단계를 마친 worktree에서 `git log --first-parent --format=%s <기준 커밋>..HEAD`는 위에서부터 다음 세 줄이다.

```
docs(handoff): record upstream import [G0-INFRA-T01]
chore(upstream): import 7-Zip 26.04 [G0-INFRA-T01]
docs(handoff): packet [G0-INFRA-T01]
```

### 7.2 구현자 단계

| 단계 | 규칙 | 출처 |
|---|---|---|
| R-1 | 시작: 곧바로 `work/handoff/G0-INFRA-T01/RESULT.md`를 상태 `PARTIAL`로 만들고 단계마다 갱신한다. 그다음 worktree를 확인한다: `git status --short --branch`가 `## chore/G0-INFRA-T01-repo-skeleton` 한 줄만 보여야 하고, 위 7.1절의 `git log --first-parent` 세 줄이 그대로 보여야 하고, `git rev-parse HEAD~3`이 0절 기준 커밋과 같아야 하고, `upstream/7zip/C/7zVersion.h`가 있어야 한다. 하나라도 다르면 ST-1. 이 카드의 어떤 단계도 네트워크를 쓰지 않는다 | [12 4.2·4.3](../../docs/12-agent-orchestration.md) |
| R-2 | 브랜치 이름이 `chore/G0-INFRA-T01-repo-skeleton`인지 확인한다. 오케스트레이터가 worktree와 함께 이미 만들었으므로 새 브랜치를 만들지 않는다 | [09 7절](../../docs/09-development-process.md), [12 4.2](../../docs/12-agent-orchestration.md) |
| R-3 | 6.7절 스크립트를 `work/evidence/G0-INFRA-T01/verify-skeleton.ps1`로 만든다(R-8의 증거 커밋에 들어간다). 폴더가 없으면 만든다 | 카드 |
| R-4 | **빨간 단계**: 스크립트를 실행해 `verify-before.log`를 남긴다. 종료 코드가 1이어야 한다. 원본은 이미 들어와 있으므로 `TC-03`·`TC-04`·`TC-06`·`TC-07`은 `PASS`, 폴더·신규 파일·문서 형식 검사(`TC-01`·`TC-02`·`TC-05`)는 `FAIL`이 정상이다 | [09 6절](../../docs/09-development-process.md) "테스트 먼저" |
| R-5 | 폴더를 만든다: 5.1절의 `.gitkeep` 17개. 그리고 `.gitignore` 끝에 `out/` 한 줄을 추가한다(파일 편집 도구 사용, LF. 이미 있는 줄은 지우지 않는다). 커밋: `chore(repo): add skeleton directories [G0-INFRA-T01]` | 6.1절 |
| R-6 | 6.3·6.5·6.6절의 파일 3개를 내용 그대로 만든다. 커밋: `docs(repo): add PATCHES, VERSIONS, NOTICES drafts [G0-INFRA-T01]` | 6.3·6.5·6.6절 |
| R-7 | **초록 단계**: 스크립트를 다시 실행해 `verify-after.log`를 남긴다. 종료 코드가 0이어야 한다. 한 줄이라도 `FAIL`이면 원인을 고치고(스크립트는 고치지 않는다) 다시 실행한다 | 카드 |
| R-8 | `EVIDENCE.md`를 10절 형식으로 작성하고 `work/evidence/G0-INFRA-T01/` 파일(스크립트, 로그 2개, 매니페스트)만 커밋한다: `chore(evidence): G0-INFRA-T01 manifest [G0-INFRA-T01]` | [09 7절](../../docs/09-development-process.md), [10 3.3](../../docs/10-logging-evidence-operations.md) |
| R-9 | `RESULT.md`를 [12 4.6](../../docs/12-agent-orchestration.md) 양식으로 마무리한다: 상태 `DONE`, 코드 커밋 SHA는 R-5·R-6 커밋을 오래된 것부터 나열, `카드와 다르게 한 점` 절에 DoD 7의 결정 3건을 적는다. **push와 PR 생성은 하지 않는다**(오케스트레이터가 한다, [12 4.3](../../docs/12-agent-orchestration.md)) | 카드 |

경계 사례와 오류 처리:

| 상황 | 처리 |
|---|---|
| R-1에서 7.1절의 세 커밋이 없거나 `upstream/7zip`이 비어 있음 | 직접 가져오려 하지 말고(네트워크 금지) ST-1로 멈춘다 |
| `TC-03`의 트리 해시가 다름 | 스크립트 상수를 고치지 않는다. ST-3 |
| `TC-04`의 줄 앵커가 다름 | 카드의 줄 번호가 틀렸다는 뜻이다. ST-4 |
| `.gitkeep`을 넣을 폴더에 이미 파일이 있음 | 그 폴더의 `.gitkeep`은 만들지 않는다(파일이 이미 있으므로). 단 `spikes/`·`tests/scratch/`에 `.gitkeep` 외 파일이 있으면 ST-5 |
| `git add`가 "LF will be replaced by CRLF" 경고를 냄 | `core.autocrlf=true` 환경의 정상 경고다. 무시하고 진행한다. 인덱스에는 LF로 들어간다(TC-05가 확인) |

## 8. 테스트 계획

테스트는 검사 스크립트(6.7절)의 `TC-nn`이다. 코드가 없는 작업이므로 단위·성능 테스트는 해당 없음 — 이유: 이 작업의 산출물은 폴더, 외부 소스, 문서다. R-3~R-4에서 스크립트를 먼저 만들어 실패를 확인한 뒤 R-5 이후 구현한다. 원본은 PRE 단계로 이미 들어와 있으므로 R-4에서 `TC-03`·`TC-04`·`TC-06`·`TC-07`은 통과하고, 구현자가 만드는 폴더·파일·문서(`TC-01`·`TC-02`·`TC-05`)만 실패한다.

| ID | 이름 | 입력 | 기대값 | 기대값 출처 | 종류 | 구성 |
|---|---|---|---|---|---|---|
| TC-01 | 폴더 20개 | 레포 작업 트리 | `src/shared`, `src/shared-win`, `src/core`, `src/app`, `src/shell`, `src/engine`, `third_party`, `tests/unit`, `tests/integration`, `tests/perf`, `tests/corpus`, `tests/scratch`, `spikes`, `tools/gate`, `tools/evidence`, `work/tasks`, `work/decisions`, `work/evidence`, `packaging`, `upstream/7zip` 20개가 모두 폴더로 존재 | [09 2절](../../docs/09-development-process.md) 구조 도표 | integration | 해당 없음(빌드 없음) |
| TC-02 | 신규 파일·`.gitignore` | 레포 작업 트리 | 파일 3개 존재, `.gitignore`에 `out/` 줄 | 5.4절 | integration | 해당 없음 |
| TC-03 | subtree 동일성 | `git rev-parse HEAD:upstream/7zip`, `git ls-files upstream/7zip`, `git status`, `C/7zVersion.h` | 트리 `2d5ed6d8d71737408bc0ddf99837171610ac215b`, 파일 1294개, 변경 0줄, `MY_VERSION_NUMBERS "26.04"` | 원본: 태그 `26.04` 가져오기 결과(1.3절 확인 방법) | integration | 해당 없음 |
| TC-04 | 줄 앵커 | `ZipItem.cpp:405`, `UTFConvert.cpp:230`, `Build.mak:80` | 405행이 `void CItem::GetUnicodeString(`로, 230행이 `bool Check_UTF8_Buf(`로 시작, 80행에 `-GR-` 포함 | 원본: 405행은 [F17 2절](../../docs/05-feature-specs/F17-filename-encoding.md), 80행은 [07 5절](../../docs/07-tech-stack.md)이 인용한 줄. 230행(`Check_UTF8_Buf` 정의)은 G2-F17-T03 카드가 인용하는 줄이며 카드 작성자가 확인함 | integration | 해당 없음 |
| TC-05 | 문서 형식 | 3개 문서 | `PATCHES.md`에 머리글 `\| 작업 ID \| 파일 \| 위치 \| 이유 \|`와 구분선 `\|---\|---\|---\|---\|`, 문서 3개 모두 BOM 없음·인덱스 줄 끝 LF | 6.3절, 카드 규칙(LF·BOM 없음, [README 2절](README.md)) | integration | 해당 없음 |
| TC-06 | subtree trailer | `git log --grep` | `git-subtree-dir: upstream/7zip` 줄과 `git-subtree-split: 9128b80e3a471108678e2b3ec3c985861c8d7b0f` 줄을 가진 커밋이 각각 1개 이상 | 6.2절(1.3절 실험 결과) | integration | 해당 없음 |
| TC-07 | 변경 범위 | `git diff --name-only <기준 커밋>..HEAD` | 모든 경로가 5.1절 허용 목록 안 | 5.1절 | integration | 해당 없음 |

## 9. 완료 조건

[09 8절](../../docs/09-development-process.md) DoD 7개 항목과 이 작업 고유 검사다.

- [ ] DoD 1 — 근거 충족: GL-1~GL-9 전부, [09 3.2](../../docs/09-development-process.md) #12. 확인: 8절 TC-01~TC-07 `ALL PASS`
- [ ] DoD 2 — 새 로직 테스트·코퍼스: 해당 없음(코드·코퍼스 없음). 대신 TC-01~TC-07
- [ ] DoD 3 — CI 3종: 해당 없음 — 이 PR 시점에 `.github/` 워크플로가 없다(G0-INFRA-T03). 그 사실을 RESULT.md에 적고 오케스트레이터가 PR 본문에 옮긴다
- [ ] DoD 4 — 증거 매니페스트: `work/evidence/G0-INFRA-T01/EVIDENCE.md`가 10절 형식으로 있고, 코드 커밋 SHA가 기록 전용 커밋 직전의 마지막 코드 커밋(R-6)과 같다. 스크립트 생성이 아니라 직접 작성한 예외라서 파일의 첫 줄이 `수동 작성(증거 도구 이전)`이다(`tools/evidence/new-manifest.ps1`은 G0-INFRA-T07 산출물이라 아직 없다. [10 3.3](../../docs/10-logging-evidence-operations.md))
- [ ] DoD 5 — 게이트 사전 검사: `tools/gate/check-task.ps1`이 없다(G0-INFRA-T10). 대신 TC-07(변경 범위)과 RESULT.md의 "남긴 임시 코드: 없음"
- [ ] DoD 6 — reviewer 승인(codex-gpt6-luna 세션), verifier 재현(`JZIP-VERIFY`): 깨끗한 클론에서 `verify-skeleton.ps1`이 `ALL PASS`
- [ ] DoD 7 — 명세와 다르게 만든 부분: 없음. 다만 아래 결정은 RESULT 보고서에 적는다 — `--squash`(6.2절), `.gitkeep` 사용과 `spikes/`·`tests/scratch/` 포함(6.1절), 매니페스트 직접 작성(DoD 4)
- [ ] DC-1 — `git diff --stat <기준 커밋>..HEAD -- upstream/7zip`에 삭제·수정 줄이 없다(추가만)
- [ ] DC-2 — (오케스트레이터) PRE-1·PRE-2 기록이 있다: `RUNLOG.md`에 판정 `채택`인 `원본 가져오기` 항목, `import-1.log`와 그 SHA-256 기록, 7.1절의 first-parent 로그 세 줄
- [ ] DC-3 — `.gitkeep` 17개가 모두 0바이트 파일이다(`spikes/`·`tests/scratch/` 포함)
- [ ] DC-4 — 문서 3개의 링크가 깨지지 않는다: `upstream/PATCHES.md`의 `../docs/09-development-process.md`, `third_party/VERSIONS.md`의 `../docs/08-architecture.md`·`../docs/09-development-process.md`·`../docs/12-agent-orchestration.md`, `upstream/PATCHES.md`의 `../docs/12-agent-orchestration.md`, `THIRD_PARTY_NOTICES.md`의 `docs/07-tech-stack.md`·`docs/10-logging-evidence-operations.md`가 레포에 존재한다
- [ ] DC-5 — `git status --short`가 비어 있다(작업 트리 깨끗)
- [ ] DC-6 — reviewer가 `THIRD_PARTY_NOTICES.md` 1절의 파일별 예외 표를 `upstream/7zip/DOC/License.txt` 8~16행과 대조해 일치함을 `REVIEW-<n>.md`의 "확인한 항목"에 적었다
- [ ] DC-7 — (병합 뒤, 오케스트레이터) 이 PR을 **병합 커밋(merge commit)** 으로 병합했고, `git log main --grep "^git-subtree-dir: upstream/7zip$" --oneline`과 `git log main --grep "^git-subtree-split: 9128b80e3a471108678e2b3ec3c985861c8d7b0f$" --oneline`이 각각 1줄 이상 나온다

## 10. 증거

| 증거 | 의미 | 명령 | 산출물 |
|---|---|---|---|
| E1' | [10 3.1](../../docs/10-logging-evidence-operations.md)의 E1은 빌드 로그지만 빌드 도구(T02)와 CI(T03)가 아직 없다. 이 카드의 E1은 **레포 골격 검사 로그**로 대신한다([10 3.2](../../docs/10-logging-evidence-operations.md) "빌드할 코드가 없는 작업"의 규칙) | R-4, R-7의 명령 | `verify-before.log`(종료 코드 1), `verify-after.log`(종료 코드 0) |
| 가져오기 기록 | 원본 가져오기(PRE-1·PRE-2)는 오케스트레이터가 실행하므로 구현자의 증거가 아니라 RUNLOG 항목이다 | PRE-1 | `work/handoff/G0-INFRA-T01/import-1.log`, `RUNLOG.md`의 `RUN-01 원본 가져오기`(오케스트레이터가 만든다) |
| E9 | 검토 기록 | reviewer 검토, verifier 재현 | `REVIEW-<n>.md`, PR 코멘트(`JZIP-VERIFY task=G0-INFRA-T01 …`) |

명령(PowerShell, 레포 루트에서):

```powershell
$T = 'work/evidence/G0-INFRA-T01'
$base = '<0절 기준 커밋 SHA>'

# R-4 (빨간 단계) — 종료 코드 1이 나와야 한다
powershell -NoProfile -File "$T/verify-skeleton.ps1" -Base $base | Out-File -Encoding ascii "$T/verify-before.log"
$LASTEXITCODE

# R-7 (초록 단계) — 종료 코드 0이 나와야 한다
powershell -NoProfile -File "$T/verify-skeleton.ps1" -Base $base | Out-File -Encoding ascii "$T/verify-after.log"
$LASTEXITCODE
```

`$base`에는 카드 0절의 기준 커밋 SHA를 쓴다. 각 `$LASTEXITCODE` 값을 `EVIDENCE.md` "실행한 명령" 표의 종료 코드 칸에 옮긴다. 원본 가져오기 명령(PRE-1)은 오케스트레이터가 실행했으므로 그 종료 코드와 소요는 `RUNLOG.md`의 `RUN-01`에서 옮긴다.

`EVIDENCE.md`(직접 작성, [10 3.3](../../docs/10-logging-evidence-operations.md) 형식). `tools/evidence/new-manifest.ps1`(G0-INFRA-T07)이 아직 없으므로 수동 작성 예외를 따른다: **파일의 첫 줄**에 `수동 작성(증거 도구 이전)`을 쓰고 그 아래 줄부터 `# 증거: G0-INFRA-T01` 제목을 쓴다.

| 매니페스트 절 | 채우는 값 |
|---|---|
| 코드 커밋 | R-6 커밋의 SHA (`git rev-parse HEAD`를 R-6 직후에 실행) |
| 환경 | `git --version`, `$PSVersionTable.PSVersion`, `[System.Environment]::OSVersion.VersionString`의 출력 |
| CI 실행 | `없음 — CI 워크플로는 G0-INFRA-T03 이후` |
| 실행한 명령 | R-4, R-7 두 줄(구성 `—`, 정의 `(없음)`, 종료 코드는 위 `$LASTEXITCODE`, 소요는 직접 잰 값)과 PRE-1 한 줄(`git subtree add …`, 오케스트레이터 실행, 값은 RUNLOG에서) |
| 산출물 | E1' 로그 2개와 `verify-skeleton.ps1`의 SHA-256(`Get-FileHash -Algorithm SHA256 <파일>`), `import-1.log`의 SHA-256(RUNLOG 값) |
| 수용 기준 대응 | GL-1→TC-01, GL-2→TC-03·TC-04, GL-3→TC-06, GL-4~GL-7→TC-02·TC-05, GL-8→TC-07, GL-9→로그 2개. 결과는 `verify-after.log`의 `PASS` 줄 번호로 적는다 |
| 외부 실행 기록, 검증, 보관본 | 머리글 줄과 구분선만 둔다(오케스트레이터가 RUNLOG에서, verifier 코멘트와 CI 봇이 행을 추가한다) |

매니페스트 경로: `work/evidence/G0-INFRA-T01/EVIDENCE.md`. 오케스트레이터는 1차 확인([12 4.9](../../docs/12-agent-orchestration.md))에서 위 명령을 직접 다시 실행하고 이 파일을 같은 형식으로 다시 쓸 수 있다(verifier 코멘트 전까지). T07이 병합된 뒤에는 이 매니페스트를 스크립트로 다시 생성하지 않는다([10 3.3](../../docs/10-logging-evidence-operations.md)).

## 11. 로그

없음 — 이 작업에는 소스 코드가 없다. 운영 이벤트 추가 0건(`EVENTS.md`는 G0-INFRA-T06에서 만든다). `JZ_DEV_TRACE`, `JZ-TEMP`, `JZ_TEST_HOOK`은 쓰지 않는다. 개인정보: 증거 로그에 사용자 이름·컴퓨터 이름이 들어가는 `Start-Transcript`를 쓰지 않는다. 로그에 홈 폴더 경로가 보이면 `<repo>`로 바꾸지 말고 RESULT에 알린다(경로 마스킹 규칙은 아직 없다).

## 12. 위험·주의

| ID | 위험 | 대처 |
|---|---|---|
| RK-1 | squash 병합하면 subtree trailer(`git-subtree-dir`, `git-subtree-split`)가 담긴 커밋이 `main`에서 사라진다. 이후 `git subtree pull`이 이전 가져오기를 찾지 못한다 | 이 PR은 병합 커밋으로 병합한다([09 7절](../../docs/09-development-process.md), DC-7). 오케스트레이터가 병합 방식을 고를 때 확인한다. `upstream/PATCHES.md` "기준" 표에도 커밋·트리 해시가 있다 |
| RK-2 | `core.autocrlf=true`인 Windows에서 7-Zip 원본의 작업 트리 파일이 CRLF가 된다. 원본 blob은 LF다 | 이 작업은 `upstream/7zip` 파일을 열거나 고치지 않는다. `git add --renormalize`를 실행하지 않는다 |
| RK-3 | 작업 트리가 깨끗하지 않은 채 `git subtree add`를 실행하면 실패한다 | 오케스트레이터가 PRE-1 시작 전에 확인한다. 가져오기는 PACKET 커밋 바로 뒤, 다른 변경보다 먼저 한다 |
| RK-4 | 빈 폴더를 만든 채 커밋하면 git이 추적하지 않는다 | `.gitkeep`을 쓴다(6.1절) |
| RK-5 | 단계 종료 검사가 `spikes/`·`tests/scratch/`의 "비어 있음"을 요구한다 | "비어 있음"은 `.gitkeep` 외 파일이 없다는 뜻이다([10 5절](../../docs/10-logging-evidence-operations.md)). 두 폴더에 `.gitkeep`만 둔다. 그 `.gitkeep`을 지우지 않는다 |
| RK-6 | Windows PowerShell 5.1이 BOM 없는 UTF-8 스크립트를 ANSI로 읽는다. `Out-File -Encoding utf8`은 BOM을 붙인다 | 스크립트는 ASCII만 쓴다. 로그는 `-Encoding ascii`로 쓴다. 문서 3개는 편집 도구로 만들어 BOM 없이 둔다(TC-05가 확인) |
| RK-7 | 함수 이름 `Git`이 `git` 명령을 가려 무한 재귀(스크립트가 멈춤) | 6.7절 스크립트처럼 `RunGit`과 `git.exe`를 쓴다 |
| RK-8 | 긴 경로: Windows 경로 한도(260자)에 가까운 곳에 worktree를 만들면 subtree 파일 쓰기가 실패할 수 있다 | 오케스트레이터가 PRE-1 전에 worktree 경로가 짧은지 본다(이 머신의 `D:/workspace/test-space/github/j-zip-wt/G0-INFRA-T01`는 짧다). 실패하면 PRE-1을 다시 한다 |
| RK-9 | `/GR-`, `/MT`·`/MD`와 LNK2038, COM STA, UI 스레드 | 해당 없음 — 컴파일하는 코드가 없다 |
| RK-10 | 줄 끝(CRLF): 문서 3개는 LF로 저장해야 한다 | TC-05가 인덱스 줄 끝을 확인한다. `.gitattributes`는 만들지 않는다(NG-19) |

## 13. 정지 조건

아래에 해당하면 추측하지 말고 멈춘 뒤 RESULT 보고서에 `status=BLOCKED`, 조건 번호, 확인한 사실, 필요한 결정을 쓴다. 만든 변경을 `main`이나 PR에 올리지 않는다.

| ID | 조건 | 공통 조건 대응 |
|---|---|---|
| ST-1 | 추적 파일이 이미 바뀌어 있거나, R-1의 이력 모양(7.1절 세 커밋, `HEAD~3` = 기준 커밋)이 다르거나, `upstream/7zip`이 비어 있거나(핸드오프 전 단계 PRE-1·PRE-2가 끝나지 않음), G0-INFRA-T16이 완료가 아니다. 구현자는 직접 가져오지 않는다 | 기준선 실패 |
| ST-2 | worktree에서 `git commit`이 막혀 있다([12 9.2](../../docs/12-agent-orchestration.md) U3·U4). 구현자는 우회하지 않고 멈춘다 | 증거 불가 |
| ST-3 | TC-03의 트리 해시나 파일 수가 다르다(태그가 옮겨졌거나 가져오기가 잘못됨). 스크립트 상수를 바꾸지 않는다 | 명세 모순 |
| ST-4 | TC-04의 줄 앵커가 하나라도 다르다. 문서가 인용한 줄 번호가 틀렸다는 뜻이다 | 명세 모순 |
| ST-5 | 5.1절 밖의 파일(특히 `upstream/7zip/**`)을 고쳐야 한다고 판단되는 경우 | 범위 밖 파일 |
| ST-6 | 6.1·6.2·6.3·6.5·6.6·6.7절의 내용을 바꿔야 한다고 판단되는 경우(예: 폴더 목록이 09 2절과 다르다) | 인터페이스 변경 |
| ST-7 | 증거 로그를 10절 방식으로 만들 수 없다(PowerShell이 없다 등) | 증거 불가 |
| ST-8 | 직접 작성하는 줄이 약 260줄(5.4절 합계)의 1.5배를 넘길 전망이다. subtree 가져온 파일은 세지 않는다 | 크기 초과 |
| ST-9 | 새 의존성, 패키지, 서브모듈, 도구 설치, 네트워크 사용이 필요하다 | 새 의존성 |
| ST-10 | 검사 스크립트를 통과시키려면 스크립트나 상수를 고쳐야 한다(테스트를 약하게 만드는 일) | 테스트 약화 |
| ST-11 | 시간 예산(S, 10분)을 넘길 것 같다. 그 시점의 상태를 `RESULT.md`에 `PARTIAL`로 남긴다 | 시간 예산 |
| ST-12 | 비밀·자격 증명이 필요하다 | 자격 증명 |

## 14. 핸드오프 메모

### 14.1 읽는 순서

1. 이 카드 전체 (특히 6절의 파일 내용, 7절의 순서)
2. [09. 구현 절차](../../docs/09-development-process.md) 2절(구조), 7절(브랜치·커밋·PR), 11절(종료 코드 규칙)
3. [10. 로그·증거·운영](../../docs/10-logging-evidence-operations.md) 3.3절(매니페스트 형식)
4. [07. 기술 스택](../../docs/07-tech-stack.md) 7절(의존성·라이선스 원칙), [08. 아키텍처](../../docs/08-architecture.md) 7.2절(공급망 행)
5. `upstream/7zip/DOC/License.txt`(PRE-1로 이미 들어와 있다. 6.6절 내용과 대조)

[12. 에이전트 오케스트레이션](../../docs/12-agent-orchestration.md)의 4.3(PACKET의 항상 허용·항상 금지·표준 규칙)과 4.6(RESULT 양식)을 따른다. 4.10(원본 가져오기)은 오케스트레이터가 읽는다.

### 14.2 산출물

| 산출물 | 이름·위치 |
|---|---|
| 브랜치 | `chore/G0-INFRA-T01-repo-skeleton` ([09 7절](../../docs/09-development-process.md) 형식 `<종류>/<작업 ID>-<짧은 이름>`) |
| 커밋 | 오케스트레이터: PACKET 커밋, PRE-1(병합 커밋 + Squashed 커밋), PRE-2 기록 커밋. 구현자: R-5, R-6, R-8. 증거 커밋은 `work/evidence/**`만 바꾼다 |
| PR | **오케스트레이터가 만든다**(구현자는 push·PR 금지). 제목 `chore(repo): add repo skeleton and 7-Zip 26.04 subtree [G0-INFRA-T01]`. 본문 재료는 RESULT.md에 있다: 근거(09 3.2 #12), 변경 요약, 증거(E1' 로그 2개, 가져오기 기록), 남긴 임시 코드 없음, 미결 사항(DoD 7의 결정 3건). **병합은 병합 커밋 방식**(squash 금지, 09 7절) |
| 증거 | `work/evidence/G0-INFRA-T01/EVIDENCE.md` + 로그 2개 + `verify-skeleton.ps1` (가져오기 기록은 `work/handoff/G0-INFRA-T01/import-1.log`, `RUNLOG.md`) |
| 보고 | `work/handoff/G0-INFRA-T01/RESULT.md` ([12 4.6](../../docs/12-agent-orchestration.md) 양식) |

### 14.3 예상 크기와 환경

- 직접 작성하는 줄: 약 260줄(5.4절). subtree 가져오기(1294개 파일)는 줄 수 기준에서 제외한다([README 6절](README.md)).
- 환경: 오케스트레이터가 만든 worktree(`../j-zip-wt/G0-INFRA-T01`, PRE-1·PRE-2가 끝난 상태), Windows, PowerShell 5.1 이상, Git for Windows(`user.name`·`user.email` 설정됨). 구현자는 네트워크를 쓰지 않는다(원본은 이미 들어와 있다). `git subtree`와 `github.com` 읽기 접속은 오케스트레이터가 PRE-1에서 쓴다.
- 소요 예상: 구현자 쪽은 검사 스크립트가 10초 미만이다(PRE-1의 `git subtree add`는 오케스트레이터가 약 8초에 끝낸다). 외부 타임아웃은 S 크기라 10분이다([12 4.4](../../docs/12-agent-orchestration.md)). 파일 편집이 대부분이므로 R-1에서 `RESULT.md`를 `PARTIAL`로 시작하고 단계마다 갱신한다.

## 개정 이력

| 개정 | 날짜 | 바꾼 절 | 이유 | 승인 |
|---|---|---|---|---|
| r0 | 2026-10-14 | 전체 | 초안 작성 | |
| r1 | 2026-10-14 | 0, 1.2, 2, 3, 4, 5, 6.1, 6.2, 6.3, 6.5, 6.7, 7, 8, 9, 10, 12, 13, 14 | 일관성 점검: 원본 가져오기를 오케스트레이터의 핸드오프 전 단계(PRE-1·PRE-2)로 이동하고 ST-2의 네트워크 조건 삭제, subtree PR은 병합 커밋으로 병합(trailer 복사 절차 삭제), 수용 기준을 09 3.2 #12로 지정, `spikes/`·`tests/scratch/`에 `.gitkeep`(폴더 18 → 20, `.gitkeep` 15 → 17), 수동 작성 매니페스트 첫 줄 규칙 | |
