# 10. 로그·증거·단계별 정리·최종 운영

코딩 에이전트가 구현하므로, **무엇을 했는지 보여주는 로그와 증거**가 완료 판단의 기준이다.
이 문서는 네 가지를 정한다.

1. 로그를 어떻게 남기는가 (운영 로그와 개발 로그의 구분)
2. 구현 중 종류별 증거를 어떻게 모으고 보관하는가
3. 단계가 끝날 때 무엇을 지우고 무엇을 남기는가
4. 출시 후 최종 운영은 어떻게 하는가

작업·단계(G0–G6) 정의는 [09. 구현 절차](09-development-process.md), 계층 구조는 [08. 아키텍처](08-architecture.md), 빌드 구성(Debug·Dev·Test·Release)은 [07. 기술 스택](07-tech-stack.md) 5절을 따른다. 사용자에게 보이는 진단 기능의 요구사항은 [OPS 진단·로그 운영](05-feature-specs/OPS-diagnostics.md)에 있다. 기술 근거는 [research/ops-research-2026-10.md](research/ops-research-2026-10.md) Q7.

---

## 1. 코드에 남는 것의 분류

모든 진단·테스트 관련 코드는 아래 다섯 종류 중 하나다. 종류마다 표시 방법과 수명이 다르다.

| 종류 | 표시 | 수명 | 배포물 포함 |
|---|---|---|---|
| **운영 로그** | `JZ_LOG_ERROR` / `WARN` / `INFO` / `VERBOSE` + 이벤트 목록 등록 | 영구 | 포함 (7절) |
| **개발 로그** | `JZ_DEV_TRACE("<작업 ID>", …)` | 해당 단계 종료까지 | 포함 안 됨 (`Dev` 구성, `JZIP_DEV_TRACE=1`에서만 컴파일) |
| **임시 코드** | `// JZ-TEMP(<작업 ID>): <이유>` … `// JZ-TEMP-END` | 해당 단계 종료까지 | 포함 안 됨 (단계 종료 시 삭제) |
| **테스트 연결점** | 포트(인터페이스) 우선. 불가피하면 `JZ_TEST_HOOK(<이름>)` | 포트는 영구 / 훅은 쓰는 테스트가 있는 동안 | 훅은 포함 안 됨 (`Test` 구성, `JZIP_TEST_HOOKS=1`에서만) |
| **테스트 코드** | `tests/unit`, `tests/integration`, `tests/perf` = 영구 · `tests/scratch` = 임시 · `spikes/` = 시제품 | 영구 / 단계 종료까지 | 포함 안 됨 |

규칙:

- 개발 로그와 임시 코드에는 **반드시 작업 ID**를 넣는다([09](09-development-process.md) 4절 형식). 그래야 누가 왜 넣었고 언제 지워야 하는지 추적된다.
- 테스트를 위해 제품 코드에 분기를 넣지 않는다. 먼저 [08](08-architecture.md) 2절의 포트(`IFileSystem`, `IClock` 등)로 해결한다.
- 측정 기준이 있는 시제품(SP-2 등)의 측정 하네스는 `spikes/`를 지우기 전에 `tests/perf/`로 옮긴다.

---

## 2. 로그 체계

### 2.1 구조

```
jzip-shared · jzip-core 코드 ─▶ ILogger 포트 (구현은 호스트 바이너리가 주입)
JZip.exe / jzip-shell.dll / 7z.exe (/ 7z.dll) ─▶ 바이너리마다 정의한 ETW TraceLogging 공급자 "JZip"
JZip.exe ─▶ 파일 로그 (LocalState\logs\)
```

| 항목 | 결정 |
|---|---|
| ETW 공급자 | 이름 `JZip`, 모든 바이너리가 같은 이름·GUID를 쓴다. GUID: G0 게이트에서 고정해 이 행에 기록한다([09](09-development-process.md) 3.2). 공급자 핸들은 모듈 범위이므로 **로그를 내는 바이너리마다** 공급자를 정의·등록한다: `JZip.exe`, `jzip-shell.dll`, `7z.exe`, 로그를 낸다면 `7z.dll`. 핸들을 다른 모듈로 넘기지 않는다(research Q7) |
| 라이브러리 | `jzip-core`·`jzip-shared`는 공급자를 정의하지 않고 `ILogger` 포트로만 로그를 낸다. 포트 구현은 그 라이브러리를 링크한 바이너리가 주입한다 |
| 수집기 없을 때 비용 | 공급자는 항상 등록한다. 수집기가 없을 때의 비용은 측정하지 않았다 [미확인] |
| 셸 확장 | `DLL_PROCESS_DETACH`에서 `TraceLoggingUnregister`를 호출한다(필수, research Q7). `DllMain` 로더 락 제약 아래에서 문제없는지 G0에서 확인한다. 셸 확장은 ETW만 쓰고 파일 로그는 쓰지 않는다 |
| 개발 로그 | `JZ_DEV_TRACE`는 `Dev` 구성에서만 컴파일되며, 내용 앞에 문자열 `JZDEV\|`를 붙여 ETW 공급자(키워드 `Dev`)와 `OutputDebugString`으로 보낸다 |
| 파일 로그 | `%LOCALAPPDATA%\Packages\<PFN>\LocalState\logs\jzip-<YYYYMMDD>-<n>.log`, 파일당 5MB, 최대 5개 순환. Info·Verbose는 **진단 모드**(7절)나 `Dev` 구성에서만 기록 |
| 오류 로그 | Error·Warning은 진단 모드와 관계없이 `LocalState\logs\errors.log`에 항상 남긴다. 1MB에서 순환, **경로·파일 이름을 담지 않는다**(이벤트 이름, 오류 코드, 확장자만) |
| 진단 모드 자동 꺼짐 | 켠 시각을 설정에 저장하고, 앱 시작 때와 실행 중 하루 한 번 검사해 7일이 지났으면 끈다 |
| 레벨 | Error · Warning · Info · Verbose. 개발 로그는 별도(`JZ_DEV_TRACE`) |
| 형식 | 이벤트 이름 + 구조화 필드. 문장형 로그 금지 (검색·비교가 쉬워야 함) |

### 2.2 운영 로그 이벤트 목록

운영 로그는 이벤트 이름이 미리 정해진 것만 쓴다. 목록은 `src/core/infra/log/EVENTS.md`에 둔다.

```markdown
| 이벤트 | 레벨 | 필드 | 언제 |
|---|---|---|---|
| Extract.Start | Info | opId, format, itemCount, mode | 해제 작업 시작 |
| Extract.End | Info | opId, result, durationMs, errorCount | 해제 작업 종료 |
| Encoding.Detected | Verbose | opId, candidate, score, confident | #17 감지 결과 |
| Shell.MenuBuilt | Verbose | itemCount, ms | S7 메뉴 생성 |
| Shell.SettingsPath | Info | source(identity/modulePath/none), pfnDerived, hasIdentity, readOk | 셸 설정 경로 결정(08 6절 ①②③, SP-1) |
```

- CI가 코드의 `JZ_LOG_*` 이벤트 이름과 목록을 대조한다. 목록에 없는 이벤트는 빌드 실패.

### 2.3 남기지 않는 것 (개인정보)

| 금지 | 대신 |
|---|---|
| 암호, 암호 관리자 내용 | 남기지 않음 |
| 아카이브 안 파일 내용 | 남기지 않음 |
| 전체 경로·파일 이름 (운영 로그) | 확장자, 경로 깊이, 길이, 개수, 크기. 같은 경로인지 구별해야 하면 **HMAC-SHA256**(설치마다 무작위로 만든 키를 `LocalState`에 두고, 진단 모드를 켤 때마다 키를 새로 만듦). 진단 모드에서 사용자가 "경로 포함"을 켠 경우만 원문 |
| 경로를 담은 설정 값 | 로그 내보내기에서 스키마의 `pii: true` 키를 뺀다("경로 포함"이 켜진 경우만 포함, [08](08-architecture.md) 6절) |
| 네트워크 전송 | 하지 않음 (원격 수집 없음) |

### 2.4 수집 방법

| 상황 | 방법 |
|---|---|
| 개발·증거 수집 | `tools/evidence/trace-start.ps1` / `trace-stop.ps1` — `tracelog`로 `JZip` 공급자를 켜고 `.etl`을 `tracefmt`로 텍스트 변환([09](09-development-process.md) 11절) |
| 사용자 지원 | 진단 모드(7절)의 파일 로그 + "로그 내보내기" |
| 크래시 | **G0 검증 후 확정.** WER `LocalDumps`가 MSIX 앱에서 동작하는지 [미확인], 파트너 센터가 MSIX 데스크톱 C++ 앱의 크래시 데이터를 보여 주는지 [미확인]. Windows App SDK 2.5.1의 매니페스트 확장 `windows.diagnosticServiceModule`(노트 문구는 .NET 앱 대상)을 평가 후보로 둔다. 결과는 G0 ADR |

---

## 3. 증거 종류와 수집 방법

### 3.1 증거 종류

| ID | 증거 | 수집 방법 | 저장 |
|---|---|---|---|
| E1 | 빌드 로그 | CI 로그 + MSBuild 바이너리 로그(`/bl`), nmake 출력 | CI 아티팩트 |
| E2 | 단위 테스트 결과 | GoogleTest `--gtest_output=xml` (JUnit XML) | CI 아티팩트 + 요약은 매니페스트 |
| E3 | 코퍼스 통합 테스트 결과 | `tests/integration` 스크립트가 아카이브별 기대값·실제값·결과 표(CSV/MD) 생성. **`Release` 바이너리로 실행** | CI 아티팩트 + 요약은 매니페스트 |
| E4 | 동작 로그 | 시나리오 실행 중 ETW 수집 → `.etl` + 변환 텍스트 | CI 아티팩트 |
| E5 | 화면 증거 | `tools/evidence/capture-window.ps1`로 창 캡처. 요소 ID(`S2-04` 등)·테마(라이트/다크)·배율(100/150/200%)을 파일 이름에 포함. GitHub 호스팅 러너는 대화형 세션이 아니어서 셸 메뉴·DPI 캡처가 확인되지 않았으므로 **로컬 PC나 대화형 셀프 호스팅 러너**에서 만든다. G3 게이트가 이 방식에 의존한다 | 축소 PNG(장당 200KB 이하)를 `work/evidence/<작업 ID>/screens/`에 커밋. 원본은 보관하지 않음 |
| E6 | 시스템 변화 | 실행 전후 파일·레지스트리 비교, REA 프로세스 동작 캡처([01](01-rea-overview.md)) | CI 아티팩트 |
| E7 | 패키지 검증 | MSIX 빌드 로그, `signtool verify /pa`, 설치 로그, `Get-AppxPackage`·매니페스트 확인, Store 제출 전 Windows App Certification Kit 보고서 | CI 아티팩트 |
| E8 | 성능 측정 | `tests/perf/run.ps1` 출력(환경: CPU·OS 빌드·아키텍처 포함) | CI 아티팩트 + 매니페스트에 수치 |
| E9 | 검토 기록 | reviewer·verifier 보고(PR 코멘트, verifier는 3.3의 고정 형식)와 외부 검토 모델의 `work/handoff/<작업 ID>/REVIEW-<n>.md`([12](12-agent-orchestration.md) 4.7) | PR + 병합 뒤 매니페스트 "보관본" 행. `REVIEW-<n>.md`는 레포(기록 전용 커밋) |
| E10 | 크래시 덤프 | **G0 검증 후 확정**(2.4) | 로컬 (매니페스트가 참조하면 업로드) |

### 3.2 작업 종류별 필수 증거

| 작업 종류 | 필수 | 해당 시 |
|---|---|---|
| 로직 (`jzip-shared`, `jzip-core`) | E1, E2, E3 | E8 |
| 포맷 핸들러 (`7z.dll`) | E1, E2, E3 | E6 |
| 화면 (`JZip.exe`) | E1, E2(PresentationModel), E5 | E4 |
| 셸 확장 (`jzip-shell.dll`) | E1, E4, E5, E6 | E7 |
| 패키징·배포 | E1, E7 | E6 |
| 시제품 (SP-*) | E4 또는 E8 + ADR. 기술 검증이 아닌 시제품(SP-6 등)은 조사 노트 + ADR. **SP-1, SP-5는 E7도 필수** | E5 |
| 모든 작업 | E9 | — |

아직 빌드할 코드가 없는 작업(예: `G0-INFRA-T01`)의 E1은 그 작업의 **검사 스크립트 실행 로그**(변경 전 실패, 변경 후 통과)로 대신할 수 있다. 어떤 로그가 E1을 대신하는지와 명령은 카드의 10절에 적는다.

### 3.3 증거 매니페스트

작업마다 `work/evidence/<작업 ID>/EVIDENCE.md`를 **스크립트로 생성**한다(`tools/evidence/new-manifest.ps1`, [09](09-development-process.md) 11절). 손으로 결과를 적지 않는다.

**예외 — 수동 작성 매니페스트**: `new-manifest.ps1`(`G0-INFRA-T07`)이 병합되기 전의 작업(`G0-INFRA-T01`~`T06` 등)은 매니페스트를 손으로 쓸 수 있다. 이 경우 파일의 **첫 줄**에 `수동 작성(증거 도구 이전)`을 적는다. 이 매니페스트는 T07이 병합된 뒤에도 스크립트로 다시 생성하지 않고 그대로 둔다(그 작업들은 이미 `JZIP-VERIFY`를 받았으므로 아래 "추가만 가능" 규칙이 적용된다). 대신 T07의 병합 PR이 이 매니페스트마다 "백필 안내" 행을 한 번 덧붙인다(내용: 수동 작성임, 스크립트로 확인한 값이 없음, 정정은 새 행으로).

```markdown
# 증거: G2-F17-T08

| 항목 | 값 |
|---|---|
| 코드 커밋 | <SHA> (기록 전용 커밋 직전의 마지막 코드 커밋) |
| 환경 | Windows 11 26200 x64, VS 2026 17.x, MSVC 14.5x |
| CI 실행 | <URL> |

## 실행한 명령
| 명령 | 구성 | 정의 | 종료 코드 | 소요 |
|---|---|---|---|---|
| tools/build.ps1 -Config Release -Arch x64 | Release | (없음) | 0 | 4m12s |
| tests/unit/run.ps1 -Config Release -Filter Encoding* | Release | (없음) | 0 | 9s |
| tests/integration/run.ps1 -Config Release -Suite encoding-score | Release | (없음) | 0 | 41s |

## 산출물
| 증거 | 파일 / 아티팩트 | SHA-256 |
|---|---|---|
| E2 | unit-encoding.xml | … |
| E3 | corpus-encoding-score.md | … |

## 수용 기준 대응
| 수용 기준 | 증거 | 결과 |
|---|---|---|
| F17 8절-1a [로직] 감지 부분 (한글 ZIP 이름 묶음, UTF-8 플래그 없음 → CP949) | E3 corpus-encoding-score.md `kr_ntfs-en` 행 | 통과 |

## 외부 실행 기록
| 역할 | 모델 | 실행 횟수 | 마지막 판정 | RUNLOG |
|---|---|---|---|---|
| 구현 | codex-gpt6-luna (gpt-6-luna) | 1 | 채택 | work/handoff/G2-F17-T08/RUNLOG.md |
| 검토 | antigravity-gemini3.8flash-medium (gemini-3.8-flash-medium) | 1 | 채택 | 같은 파일 |

## 검증
| 역할 | 결과 | 시각 | 출처 |
|---|---|---|---|
| verifier | 재현 성공 | … | PR #<번호> 코멘트 |

## 보관본
| PR | main 커밋 (squash 또는 병합 커밋) | 릴리스 자산 | 추가 시각 |
|---|---|---|---|
| #<번호> | <SHA> | gate-G<n> / evidence-pr-<번호>.zip | … |
```

규칙:

- **코드 커밋 SHA**: 매니페스트에는 기록 전용 커밋 직전의 마지막 코드 커밋 SHA를 적는다. **기록 전용 커밋**은 `work/handoff/**` 또는 `work/evidence/**`만 바꾸는 커밋이다([09](09-development-process.md) 7절, [12](12-agent-orchestration.md) 4.5). 기록 전용 커밋에서는 경로 필터 작업이 빌드·테스트 작업을 **작업(job) 단위로** 건너뛰고 성공으로 보고한다(워크플로 단위 `paths-ignore`는 필수 검사를 대기 상태로 남기므로 쓰지 않는다). CI는 매니페스트의 SHA가 "PR 머리에서 기록 전용 커밋을 뺀 커밋"과 같은지 검사한다([09](09-development-process.md) 7절).
- **외부 실행 기록 절**: `work/handoff/<작업 ID>/RUNLOG.md`가 있으면 `new-manifest.ps1`이 위 예시의 "외부 실행 기록" 절(역할별 모델, 실행 횟수, 마지막 판정, RUNLOG 링크)을 RUNLOG에서 생성한다. RUNLOG가 없으면 절을 쓰지 않는다. 이 기능은 `G0-INFRA-T07` 범위다.
- **병합 뒤 추적**: squash 병합하면 매니페스트의 코드 커밋 SHA는 `main`에 남지 않지만 `refs/pull/<번호>/head`로 계속 도달할 수 있다. 병합 커밋 방식(subtree를 추가·갱신하는 PR, [09](09-development-process.md) 7절)은 코드 커밋이 `main` 이력에 그대로 남는다. CI 봇은 어느 방식이든 "보관본" 행에 `main`의 병합 SHA를 함께 적는다.
- **명령 행**: 명령마다 빌드 구성과 정의(`JZIP_DEV_TRACE`, `JZIP_TEST_HOOKS`)를 적는다. 테스트 훅에 의존하는 묶음은 `Test` 구성으로, E3 코퍼스 증거는 `Release` 바이너리로 실행한다.
- **verifier 결과**: verifier는 PR 코멘트에 아래 고정 형식으로 쓴다. 병합 뒤 CI 작업(봇)이 이 코멘트를 읽어 "검증" 행과 "보관본" 행을 매니페스트에 덧붙인다. 이 봇 커밋이 `main`에 허용된 유일한 자동 커밋이며 `work/evidence/**`만 바꾼다.

  ```
  JZIP-VERIFY task=<작업 ID> code=<SHA> result=pass|fail
  commands=<재실행한 명령 수> mismatches=<불일치 수>
  note=<한 줄>
  ```

- **추가만 가능 (verifier 코멘트 이후)**: verifier의 `JZIP-VERIFY` 코멘트가 달리기 전까지는 implementer가 매니페스트를 스크립트로 다시 생성할 수 있다(예: 리뷰 지적을 고친 뒤). 그 코멘트가 달린 뒤에는 기존 행과 링크를 고치거나 지우지 않고 새 행만 덧붙인다. 정정도 새 행으로 한다.
- **보관**: 단계를 시작할 때 gatekeeper(CI 작업)가 초안 릴리스 `gate-G<n>`을 만든다. 그 단계의 PR이 병합될 때마다 CI가 그 PR의 아티팩트를 묶어 `evidence-pr-<번호>.zip`으로 이 초안 릴리스에 자산으로 붙인다(증분). 단계 게이트를 통과하면 릴리스를 게시한다. CI 아티팩트는 `retention-days: 90`을 명시한다.
- 큰 파일(`.etl`, 덤프, 바이너리 로그)은 레포에 커밋하지 않는다. 레포에 들어가는 이진 파일은 E5 축소 PNG뿐이다.
- 수용 기준 하나에 증거가 하나 이상 연결되지 않으면 작업은 완료가 아니다.

---

## 4. 구현 중 임시 코드 관리

| 항목 | 규칙 |
|---|---|
| 개발 로그 | `JZ_DEV_TRACE("G2-F17-T03", "score", candidate, score)` — 작업 ID 필수. `Dev` 구성에서만 `JZIP_DEV_TRACE=1` |
| 임시 코드 | `// JZ-TEMP(G1-SP2-T01): 10만 항목 생성용 하드코딩` … `// JZ-TEMP-END` |
| 테스트 훅 | `JZ_TEST_HOOK(ForceDiskFull)` — `Test` 구성(`JZIP_TEST_HOOKS=1`)에서만 컴파일. 같은 목적을 포트로 해결할 수 있으면 훅 금지 |
| 탐색용 테스트 | `tests/scratch/<작업 ID>/` — 다른 테스트가 의존하지 않음 |
| 시제품 | `spikes/<SP 번호>/` — 제품 프로젝트에서 참조 금지 |

PR 검사(`tools/gate/check-task.ps1`):

- 새로 추가된 `JZ_DEV_TRACE`·`JZ-TEMP`·`JZ_TEST_HOOK`·`JZIP-PATCH`가 작업 ID 형식([09](09-development-process.md) 4절 정규식)을 지키는지
- `JZIP-PATCH`가 작업 카드의 "허용 원본 파일"에만 있고 `upstream/PATCHES.md`에 기록됐는지
- `Release` 구성에 `JZIP_DEV_TRACE`·`JZIP_TEST_HOOKS`가 정의되지 않았는지 — `.vcxproj`(MSBuild)와 `tools/build.ps1`의 nmake 호출 **양쪽**을 검사
- 제품 프로젝트가 `spikes/`·`tests/scratch/`를 참조하지 않는지, `jzip-shell`이 `jzip-core`·WinRT를 링크하지 않는지
- PR 본문의 "남긴 임시 코드" 목록과 실제 표시가 일치하는지

---

## 5. 단계 종료 정리

단계(G0–G6)를 끝낼 때 gatekeeper 에이전트가 정리 작업 `G<n>-CLEANUP-T01`을 만들어 아래를 처리한다.

| 대상 | 처리 | 확인 |
|---|---|---|
| 개발 로그 `JZ_DEV_TRACE` | 삭제. 계속 필요한 정보면 운영 로그로 **승격**(이벤트 목록 등록 후 `JZ_LOG_*`로 교체) | 해당 단계 작업 ID로 검색 0건 |
| 임시 코드 `JZ-TEMP` | 삭제하거나 정식 코드로 바꾸고 표시 제거 | 검색 0건 |
| 테스트 훅 | 사용하는 테스트가 없으면 삭제. 남는 훅은 포트로 바꿀 수 있는지 검토 | 미사용 훅 0건 |
| `tests/scratch/` | 가치 있는 테스트는 `tests/unit`·`tests/integration`으로 옮기고(기대값을 명세 기준으로 다시 확인), 나머지 삭제 | 폴더 비어 있음(`.gitkeep` 외 파일 없음) |
| `spikes/` | 측정 하네스는 `tests/perf/`로 승격, ADR 작성 후 삭제. 기록용으로 git 태그 `spike/<SP 번호>-final` | 폴더 비어 있음(`.gitkeep` 외 파일 없음), ADR 존재 |
| 증거 산출물 | PR마다 이미 `evidence-pr-<번호>.zip`으로 초안 릴리스 `gate-G<n>`에 붙어 있음(3.3). 게이트 통과 때 이 릴리스를 게시 | 단계의 모든 매니페스트에 "보관본" 행이 있고 링크가 유효 |
| 로컬 진단 파일 | 어떤 매니페스트도 참조하지 않는 `.etl`, 덤프, 캡처 원본은 삭제. 매니페스트가 참조하는 항목은 업로드가 끝난 뒤에만 삭제 | — |
| 증거 수집 도구 `tools/evidence` | **유지** (배포물에 포함 안 됨) | — |
| 영구 테스트(`unit`·`integration`·`perf`)·코퍼스·운영 로그·로그 기반 코드 | **유지** | — |

"비어 있음"은 `.gitkeep`을 뺀 파일이 없다는 뜻이다. `spikes/`와 `tests/scratch/`의 `.gitkeep`은 `G0-INFRA-T01`이 만들며 정리에서도 지우지 않는다(폴더가 레포에서 사라지지 않게 한다).

정리가 끝나면 `tools/gate/check-stage.ps1 -Stage G<n>`을 실행한다. 이 스크립트의 보고서가 단계 종료 증거다. **gatekeeper가 아닌 verifier**가 `check-stage`를 다시 실행해 그 결과를 `G<n>-CLEANUP-T01` 카드의 증거로 남긴다.

- 단계 작업 전체가 완료(DoD)인지 — `G<n>-CLEANUP-T01` 카드 자체는 이 검사에서 제외
- 위 표의 확인 항목
- `Release` 구성 MSIX에서 꺼낸 `JZip.exe`, `jzip-shell.dll`, `7z.dll`, `7z.exe`에 개발 로그 표시 문자열 `JZDEV|`와 작업 ID 패턴(`G[0-6]-…-T\d{2}`)이 없는지 바이너리 검사
- 운영 로그 이벤트 목록과 코드 일치
- `upstream/PATCHES.md`와 실제 `JZIP-PATCH` 표시 일치

---

## 6. 출시 체크리스트 (G6)

- [ ] 모든 단계의 정리 보고서(`check-stage`) 통과
- [ ] 코드 전체에 `JZ_DEV_TRACE`, `JZ-TEMP`, 미사용 `JZ_TEST_HOOK` 0건
- [ ] `Release` 구성 확인(MSBuild·nmake 양쪽), 바이너리 4종에 `JZDEV|`·작업 ID 문자열 없음
- [ ] 회귀 코퍼스 전체 통과(Release), 성능 기준(SP-2 등, `tests/perf`) 충족
- [ ] MSIX 서명 검증, 설치·제거 후 시스템 변화(E6) 확인
- [ ] Windows App Certification Kit 통과 (Store)
- [ ] `THIRD_PARTY_NOTICES.md`, LGPL 소스 공개 링크, `upstream/PATCHES.md` 최신
- [ ] 운영 로그에 개인정보 금지 항목(2.3) 위반 없음 — 이벤트 목록 필드 검토, `errors.log`에 경로 없음
- [ ] 단계별 증거 보관본(`gate-G*` 릴리스의 `evidence-pr-*.zip`) 링크 유효

---

## 7. 최종 운영 방식 (출시 후)

| 항목 | 운영 방식 |
|---|---|
| 기본 상태 | ETW 공급자 등록, Error·Warning만 `errors.log`(경로 없음)에 기록, Info·Verbose 파일 로그 꺼짐. 원격 전송(텔레메트리) 없음 |
| 진단 모드 | 설정 S8-22 "진단 로그 기록"을 켜면 Info·Verbose 파일 로그 기록. **7일 뒤 자동으로 꺼짐**(앱 시작 + 하루 한 번 검사). "경로 포함"은 별도 체크(기본 꺼짐). 켤 때 경로 HMAC 키를 새로 만듦 |
| 로그 내보내기 | S8-23 "로그 내보내기…" → 파일 로그 + `errors.log` + 설정 파일(암호 관련 항목 제외, `pii: true` 키는 "경로 포함"일 때만) + 앱·커밋·7-Zip·Windows App SDK·OS 버전을 zip 하나로 저장. 사용자가 직접 이슈에 첨부 |
| 크래시 | **G0 검증 후 확정**(2.4). 확정 전 안내: 사이드로드 사용자에게 진단 모드 + 재현 + 내보내기를 요청한다. 덤프를 받아야 하면 고급 사용자 문서로 `DumpType=1`(미니 덤프)만 안내하고, 덤프에 암호 등 비밀이 들어 있을 수 있으므로 공개 이슈에 올리지 말고 비공개 경로로 보내도록 경고한다 |
| 버전 표시 | 정보 화면에 앱 버전, 커밋 SHA, 7-Zip 기준 버전, Windows App SDK 버전 |
| 문제 대응 흐름 | 이슈 접수 → 내보낸 로그로 재현 → 회귀 코퍼스에 재현 아카이브 추가(가능하면) → 수정 작업 카드 → 일반 절차([09](09-development-process.md)) |
| 운영 로그 변경 | 이벤트 추가·변경은 이벤트 목록 갱신 + 개인정보 검토(2.3)를 같은 PR에서 |
