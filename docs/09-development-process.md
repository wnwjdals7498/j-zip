# 09. 구현 절차와 코딩 에이전트 운영

j-zip은 코딩 에이전트가 구현한다. 그래서 **무엇을 근거로 만들었고, 어떻게 확인했는지**가 코드만큼 중요하다.
이 문서는 작업 단위, 단계(G0–G6, G = 게이트)와 종료 조건, 에이전트 역할, 브랜치·커밋·PR 규칙, 도구 계약을 정한다.
로그·증거 수집과 단계 종료 시 정리 규칙은 [10. 로그·증거·운영](10-logging-evidence-operations.md), 구조와 패턴은 [08. 아키텍처](08-architecture.md)를 따른다.

## 1. 원칙

| 원칙 | 내용 |
|---|---|
| 명세 주도 | 모든 작업은 [05. 기능 세부 명세](05-feature-specs/README.md)의 요구사항 ID(`FR-17-5` 등)나 [06. GUI 명세서](06-gui-spec.md)의 요소 ID(`S2-04` 등)에서 출발한다. 명세에 없는 동작을 만들지 않는다 |
| 증거로 완료 | "완료"는 주장이 아니라 증거다. 수용 기준(명세 8절)마다 정해진 종류의 증거가 있어야 완료로 본다 |
| 작성과 검토 분리 | 코드를 쓴 에이전트는 자기 작업을 승인하지 않는다. 검토와 검증은 별도 에이전트(별도 컨텍스트)가 한다 |
| 작게 | 작업 하나 = PR 하나 = 리뷰 가능한 크기(대략 변경 400줄 이하, 생성 파일 제외) |
| 사실만 | 7-Zip 내부 동작은 [research](research/7zip-26.04-source-facts.md)나 실제 소스로 확인한 것만 근거로 쓴다. 모르면 "미결"로 남기고 결정 기록(ADR)을 요청한다 |

---

## 2. 레포 구조

```
j-zip/
├─ upstream/
│  ├─ 7zip/              7-Zip 원본 (git subtree prefix, 26.04부터)
│  └─ PATCHES.md         j-zip이 원본 파일에 넣은 패치 목록 (subtree prefix 밖)
├─ src/
│  ├─ shared/            jzip-shared.lib — 순수 도메인 로직 (표준 C++20, OS 호출 없음, 08 3절)
│  ├─ shared-win/        jzip-win.lib    — jzip-shared 포트의 Win32 구현 (WinRT·XAML 없음, 08 3절)
│  ├─ core/              jzip-core.lib   — 응용·기반 계층 (UI 없음)
│  ├─ app/               JZip.exe        — WinUI 3 화면
│  ├─ shell/             jzip-shell.dll  — 우클릭 메뉴 (jzip-shared·jzip-win만 링크)
│  └─ engine/            7z.dll·7z.exe 빌드 래퍼(nmake) + 추가 포맷 핸들러(ALZ, LZ4)
├─ third_party/          lz4, unalz (각 LICENSE 포함)
├─ tests/
│  ├─ unit/              GoogleTest — 영구
│  ├─ integration/       코퍼스 기반 통합 테스트 — 영구
│  ├─ perf/              성능 측정 하네스 — 영구 (SP-2 등에서 승격)
│  ├─ corpus/            샘플 아카이브 (작은 파일만 커밋, 큰 파일은 생성 스크립트)
│  └─ scratch/           탐색용 테스트 — 단계 종료 시 정리
├─ spikes/               시제품 코드 — 단계 종료 시 삭제
├─ tools/
│  ├─ build.ps1          빌드 진입점 (MSBuild + nmake, 11절)
│  ├─ gate/              작업·단계 검사 스크립트
│  └─ evidence/          증거 수집 스크립트 (배포물에 포함 안 됨)
├─ work/
│  ├─ tasks/             작업 카드
│  ├─ decisions/         결정 기록 (ADR)
│  └─ evidence/          증거 매니페스트 (텍스트 + 축소 화면 PNG만, 큰 파일은 CI 아티팩트·릴리스 자산)
├─ packaging/            MSIX 매니페스트, 서명 설정
├─ docs/, design/
└─ jzip.sln
```

7-Zip 원본 수정 규칙:

- 원본 파일을 고쳐야 하면 바뀐 부분을 `// JZIP-PATCH(<작업 ID>): <이유>` … `// JZIP-PATCH-END` 로 감싸고 `upstream/PATCHES.md`에 한 줄로 기록한다.
- 원본 파일은 작업 카드의 "허용 원본 파일" 행에 **파일 단위로 명시된 것만** 고칠 수 있다. 목록에 없는 원본 파일을 고쳐야 하면 작업을 멈추고 메모에 기록한다.
- 원본 패치는 `jzip-shared`·`jzip-win` 호출(훅)과 그 결과 전달에 그친다. 판단 로직 자체는 `src/shared/`, Win32 포트 구현은 `src/shared-win/`에 둔다([08](08-architecture.md) 1절).
- 가능하면 원본은 고치지 않고 `src/` 쪽에 새 파일을 만든다. 패치는 업스트림 갱신 때 충돌 지점이 된다.

---

## 3. 단계(G0–G6)와 종료 조건

### 3.1 단계 표

| 단계 | 내용 | 종료 조건 (게이트) |
|---|---|---|
| **G0 골격** | 레포 구조, 7-Zip subtree, `jzip-shared`·`jzip-win` 빈 라이브러리(`-mt`·`-md` 두 변형) + nmake 연결, MSBuild 솔루션, CI 3종, 빌드 구성 4종([07](07-tech-stack.md) 5절), 로그 뼈대([10](10-logging-evidence-operations.md)), 테스트 프레임워크, 코퍼스 v0, 11절 도구 | 3.2의 G0 게이트 항목 전부 |
| **G1 시제품** | [07](07-tech-stack.md) 9절의 SP-1~SP-6 (`spikes/`에서) | 각 시제품의 결론을 ADR로 기록(필수 증거는 [10](10-logging-evidence-operations.md) 3.2), 측정 기준이 있는 시제품의 하네스를 `tests/perf/`로 승격, `spikes/` 삭제 |
| **G2 로직 P1** | `jzip-shared`·`jzip-win`·`jzip-core` + CLI로 P1 로직(3.3 표의 G2 행). 진입 조건은 표 아래 | 해당 기능의 `[로직]` 수용 기준을 CLI·단위·코퍼스 테스트로 증명. `[화면]` 수용 기준은 G3에서 증명 |
| **G3 기본 화면** | `JZip.exe` S1·S2·S3·S4·S5·S9, `jzip-shell` S7, MSIX 패키지와 셸 등록(#19의 매니페스트 `comServer`·`windows.fileExplorerContextMenus` 선언, [F19](05-feature-specs/F19-win11-menu.md) 3절의 G3 FR), #32, 기존 기능 #10·#14·#18·#20·#35의 WinUI 재구현 | 설치 → 우클릭 → 풀기 → 알림까지 전체 흐름 증거(E5는 대화형 러너에서, [10](10-logging-evidence-operations.md) 3.1). G2 기능의 `[화면]` 수용 기준 증거 |
| **G4 P2** | P2 기능(#19는 S8-04·S8-18 화면과 메뉴 방식 설정만) + S6, S8, D1–D3, #22, S8-22·S8-23·S8-24(OPS) | 기능별 수용 기준 증거 |
| **G5 P3·배포** | P3 기능, 서명, Store 제출, WinGet | 서명된 MSIX 설치 검증, Store 인증 통과 |
| **G6 출시 후보** | 회귀 전체, 성능 측정, 문서 정리 | 출시 체크리스트([10](10-logging-evidence-operations.md) 6절) 통과 |

단계를 넘어가기 전에 **단계 종료 정리**([10](10-logging-evidence-operations.md) 5절)를 반드시 한다.

단계를 시작할 때:

- gatekeeper(CI 작업)가 초안 릴리스 `gate-G<n>`을 만든다. 그 단계 PR의 증거 보관본 `evidence-pr-<번호>.zip`은 병합될 때마다 이 릴리스에 붙고, 게이트를 통과하면 릴리스를 게시한다([10](10-logging-evidence-operations.md) 3.3).
- G2 진입 조건: **CLI 스위치 이름 결정 ADR**이 채택 상태여야 한다. 대상은 [F13](05-feature-specs/F13-smart-extract.md) FR-13-12(스마트 해제 스위치)와 [F31](05-feature-specs/F31-recycle-after.md) FR-31-10(해제 후 휴지통 스위치)의 미결 이름이다.

### 3.2 G0 게이트 항목

작업 카드는 이 표의 번호로 항목을 가리킨다(예: `09 3.2 #3`, 8절).

| # | 항목 | 확인 |
|---|---|---|
| 1 | 기준선 빌드 | 원본과 같은 동작의 `7z.exe`·`7z.dll`이 CI 3종에서 빌드되고 코퍼스 v0 테스트 통과 |
| 2 | `jzip-shared`·`jzip-win` 연결 | MSBuild로 만든 `jzip-shared-mt.lib`·`jzip-win-mt.lib`를 nmake makefile 변수(`JZIP_SHARED_LIB`, `JZIP_WIN_LIB`)로 `7z.dll`·`7z.exe`가 링크([07](07-tech-stack.md) 5절) |
| 3 | CRT·플래그 일치 | LNK2038 0건, 두 변형(`-mt`·`-md`) 모두 빌드. `-mt` 변형이 [07](07-tech-stack.md) 5절 플래그 계약(`/MT`, `/EHsc`, `/GR-`, `/Gy /GF`, `/GL` 없음, 모든 구성에서 `_ITERATOR_DEBUG_LEVEL=0`)을 따름 |
| 4 | 빌드 구성 | Debug·Dev·Test·Release 4종이 MSBuild와 nmake(`tools/build.ps1 -Config`) 양쪽에서 동작 |
| 5 | 셸 링크 검사 | CI가 `jzip-shell.dll` 링크 입력에 `jzip-core.lib`와 WinRT 라이브러리가 없음을 검사([08](08-architecture.md) 1절) |
| 6 | 코퍼스 v0 | 최소 세트: 한글 이름 ZIP(UTF-8 플래그 있음/없음), 분할(`.zip`+`.z01`, `.7z.001`, RAR `.partN.rar`), 암호 ZIP·7z, 손상 ZIP, `tar.gz`·`tgz`·`tar.xz`. `tests/corpus/generate.ps1`로 만들고 정답 파일 포함 |
| 7 | ETW 공급자 | 공급자 GUID를 고정해 [10](10-logging-evidence-operations.md) 2.1에 기록. 셸 DLL의 `DLL_PROCESS_DETACH`에서 `TraceLoggingUnregister` 호출이 로더 락 아래에서 문제없는지 확인 |
| 8 | ADR: LocalDumps | WER LocalDumps가 MSIX 패키지 앱(`JZip.exe`)에서 덤프를 남기는지 실측한 결과 |
| 9 | ADR: C++/WinRT 버전 | Windows App SDK 2.5 템플릿이 실제로 참조하는 C++/WinRT 버전을 확인하고, [07](07-tech-stack.md)에 고정한 3.0.x와의 관계를 기록 |
| 10 | 도구 | 11절 도구가 계약대로 동작(각 도구의 종료 코드 0/1/2 확인) |

### 3.3 기능·화면 ↔ 단계

G2는 `jzip-shared`·`jzip-core` 로직과 CLI, G3는 기본 화면, G4는 P2, G5는 P3와 배포다. S8 화면은 G4에서 만들며, 그 전 단계 기능은 설정 기본값으로 동작한다. 화면 요소 중 뒤 단계 기능에 속한 요소(예: S9-03 #21, S9-05 #24, S8-20 #36)는 그 기능의 단계에서 추가한다.

| # | 기능 | 순위 | 로직 | 화면 |
|---|---|---|---|---|
| 1 | F01 분할 압축 | — | G3 | G3 (S3) |
| 2 | F02 4GB 넘는 아카이브 | — | G3 | G3 (S3) |
| 3 | F03 멀티코어 압축 | — | G3 | G3 (S3) |
| 4 | F04 AES-256 암호 | — | G3 | G3 (S3), G4 (S6) |
| 5 | F05 추가 압축 포맷 생성 | P3 | G5 | G5 |
| 6 | F06 자동 해제 EXE | P3 | G5 | G5 |
| 7 | F07 ALZ 해제 | P2 | G4 | G4 |
| 8 | F08 ZSTD/LZ4 해제 | P3 | G5 | G5 |
| 9 | F09 ACE 해제 | 제외 | — | — |
| 10 | F10 풀지 않고 목록 보기 | — | G3 | G3 (S1) |
| 11 | F11 드래그해서 꺼내기 | P2 | G4 | G4 |
| 12 | F12 TGZ/TBZ 한 번에 풀기 | P1 | G2 | G3 (S2-08) |
| 13 | F13 스마트 해제 | P1 | G2 | G3 (S2, S7) |
| 14 | F14 풀지 않고 아카이브 수정 | — | G3 | G3 (S1) |
| 15 | F15 아카이브 안 파일 바로 편집 | P2 | G4 | G4 (S1-20, D1) |
| 16 | F16 주석 보기 | P3 | G5 | G5 |
| 17 | F17 파일명 인코딩 자동 감지 | P1 | G2 | G3 (S1-12, S5, S2-10) |
| 18 | F18 우클릭 메뉴 | — | G3 | G3 (S7) |
| 19 | F19 Win11 새 컨텍스트 메뉴 | P2 | G3 (MSIX 등록·메뉴 동작) | G3 (S7), G4 (S8-04, S8-18) |
| 20 | F20 무결성 테스트 | — | G3 | G3 (S9) |
| 21 | F21 악성코드 검사 | P3 | G5 | G5 |
| 22 | F22 다국어 | — | G4 | G4 (S8-15) |
| 23 | F23 비밀번호 관리자 | P2 | G4 | G4 (S6, D2) |
| 24 | F24 손상 ZIP 복구 | P3 | G5 | G5 (S9-05, D3 연결) |
| 25 | F25 비밀번호 찾기 | 제외 | — | — |
| 26 | F26 이미지 썸네일 | P2 | G4 | G4 |
| 27 | F27 ARM64·스토어 배포 | P3 | G3 (MSIX) | G5 (서명·Store·WinGet) |
| 28 | F28 다크모드 | P2 | G4 | G4 |
| 29 | F29 고DPI·모던 아이콘 | P2 | G4 | G4 |
| 30 | F30 해제 후 폴더 열기 | P1 | G2 | G3 (S2-06, S4) |
| 31 | F31 해제 후 원본 휴지통 | P1 | G2 | G3 (S2-07, S4) |
| 32 | F32 자동 닫기·완료 알림 | P1 | G3 | G3 (S4) |
| 33 | F33 분할 파일 자동 인식 | P2 | G4 | G4 |
| 34 | F34 메뉴 항목 선택 | P3 | G5 | G5 (S8-05) |
| 35 | F35 여러 개 각각 폴더로 풀기 | — | G2 (#13 규칙) | G3 (S7-10) |
| 36 | F36 탭 탐색기 | P3 | G5 | G5 (S1-01, S8-20) |
| OPS | [진단·로그 운영](05-feature-specs/OPS-diagnostics.md) | — | G0 (로그 뼈대) | G4 (S8-22, S8-23, S8-24) |

| 화면 | 단계 | 화면 | 단계 |
|---|---|---|---|
| S1 메인 창 | G3 | S6 암호 입력 / 관리자 | G4 |
| S2 압축 풀기 | G3 | S7 우클릭 메뉴 | G3 |
| S3 새로 압축 | G3 | S8 설정 | G4 |
| S4 진행창 | G3 | S9 테스트 결과 | G3 |
| S5 파일명 인코딩 | G3 | D1 · D2 · D3 | G4 (D3의 복구 동작은 #24와 함께 G5에서 연결) |
| S8-22 · S8-23 · S8-24 (OPS) | G4 | | |

### 3.4 수용 기준 태그

05 명세 8절의 수용 기준은 `[로직]`(CLI·단위·코퍼스 테스트로 증명) 또는 `[화면]`(화면·셸 증거로 증명) 태그를 단다([05 양식](05-feature-specs/README.md) 8절). 36개 명세를 한꺼번에 다시 태그하지 않고, **그 기능의 작업 카드를 만들 때** 명세에 태그를 달고 작업 카드는 그 태그를 그대로 옮긴다. G2 게이트는 `[로직]` 기준만, G3 이후는 `[화면]` 기준까지 본다.

한 기준에 로직과 화면이 섞여 있으면 planner는 명세 PR에서 그 기준을 하위 항목 `1a [로직]` / `1b [화면]`으로 나눌 수 있다. 의미를 바꾸지 않는 분할이므로 ADR은 필요 없다. 예: [F17](05-feature-specs/F17-filename-encoding.md) 8절 1a·1b, 2a·2b, 3a.

---

## 4. 작업 카드

작업은 `work/tasks/<작업 ID>.md` 파일로 만든다. 에이전트는 이 파일만 보고 작업을 시작할 수 있어야 한다.

작업 ID 규칙: `<단계>-<분류>-T<번호>`. 정규식 `^G[0-6]-(F\d{2}|SP\d|INFRA|OPS|CLEANUP|UPSTREAM)-T\d{2}$`.

- 기능 번호는 두 자리로 쓴다(`F07`). 예) `G2-F17-T03`, `G4-F07-T01`
- 시제품은 `SP1`~`SP6`(본문에서는 "SP-1"처럼 써도 된다). 예) `G1-SP2-T01`
- 기능과 무관한 작업은 `G0-INFRA-T01`, 운영 기능은 `G4-OPS-T01`, 단계 정리는 `G3-CLEANUP-T01`, 업스트림 갱신은 `G5-UPSTREAM-T01`

```markdown
# G2-F17-T03 인코딩 후보 점수 계산

| 항목 | 내용 |
|---|---|
| 단계 | G2 |
| 근거 | F17 FR-17-5 ~ FR-17-10 |
| 수용 기준 | F17 8절 1a·2a·3a [로직] |
| 변경 범위 | src/shared/encoding/** , tests/unit/encoding/** |
| 허용 원본 파일 | upstream/7zip/CPP/7zip/Archive/Zip/ZipItem.cpp (JZIP-PATCH 훅만) |
| 금지 범위 | 위에 없는 upstream/** (필요하면 작업 중단 후 보고), src/app/** |
| 선행 작업 | G2-F17-T02 |
| 필요한 증거 | E1, E2, E3, E9 |
| 작성 | implementer 에이전트 |
| 검토 | reviewer 에이전트 → verifier 에이전트 |

## 할 일
## 완료 조건
## 메모 (작업 중 발견한 사실, 미결 사항)
```

`근거`·`수용 기준` 행: 기능 카드는 명세의 FR과 8절 항목(태그 포함)을 적는다. FR이 없는 카드는 아래를 수용 기준으로 적는다(8절).

| 카드 분류 | `수용 기준` 행에 적는 것 |
|---|---|
| `G0-INFRA` | 3.2 G0 게이트 항목 번호 (예: `09 3.2 #2, #3`) |
| `CLEANUP` | [10](10-logging-evidence-operations.md) 5절 표의 행 (대상 이름) |
| `UPSTREAM` | 10절 단계 번호 |
| `OPS` | [OPS 진단](05-feature-specs/OPS-diagnostics.md)의 FR-OPS와 8절 수용 기준 |

---

## 5. 에이전트 역할

| 역할 | 하는 일 | 하지 않는 일 |
|---|---|---|
| planner | 명세에서 작업 카드 생성(수용 기준 태그 포함), 순서·선행 관계 정리 | 코드 작성 |
| implementer | 작업 카드 범위 안에서 테스트와 코드 작성, `tools/evidence/new-manifest.ps1`로 증거 매니페스트 생성 | 범위 밖 수정, 자기 PR 승인, 테스트 비활성화 |
| reviewer | 변경 검토(명세 일치, 결함, 단순화, 금지 패턴). 결과는 PR 코멘트(E9) | 코드 수정 (지적만) |
| verifier | 깨끗한 환경에서 빌드·테스트·게이트를 다시 실행해 증거가 재현되는지 확인. 결과를 정해진 형식의 PR 코멘트(E9, [10](10-logging-evidence-operations.md) 3.3)로 남김. 단계 정리 카드(`G<n>-CLEANUP-T01`)에서는 `check-stage`를 다시 실행 | 증거 없이 승인 |
| gatekeeper | 단계 시작 때 초안 릴리스 `gate-G<n>` 생성(CI 작업), 단계 종료 정리 카드 생성과 정리 실행, 게이트 통과 때 릴리스 게시 | 기능 추가, 자기 정리 결과 승인 |
| CI 봇 | 병합 뒤 verifier 결과를 매니페스트에 "검증"·"보관본" 행으로 덧붙이는 커밋(`work/evidence/**`만). "보관본" 행에 `main`의 squash 병합 커밋 SHA를 적는다 | 그 밖의 모든 변경 |

규칙:

- implementer와 reviewer·verifier는 **서로 다른 세션**에서 실행한다.
- 에이전트가 막히면(명세 모순, 허용되지 않은 원본 패치 필요, 외부 결정 필요) 추측하지 말고 작업 카드 "메모"에 기록하고 멈춘다. 사람이 결정하면 ADR을 남긴다.

---

## 6. 구현 순서 (작업 하나)

```
작업 카드 읽기
 → 관련 명세(05, 06, 07, 08)와 research 확인
 → 실패하는 테스트 먼저 작성 (변경 경계에서)
 → 구현
 → 로컬 빌드·테스트 실행 (tools/build.ps1, tests/*/run.ps1)
 → 코드 커밋 → 증거 매니페스트 생성 (tools/evidence/new-manifest.ps1) → 매니페스트만 따로 커밋
 → 게이트 사전 검사 (tools/gate/check-task.ps1)
 → PR 생성 (증거 섹션 포함)
 → reviewer 검토 → 수정 (수정하면 매니페스트를 새로 생성 — verifier 코멘트 전까지만 허용)
 → verifier 재현, PR 코멘트(JZIP-VERIFY)로 결과 기록 — 이후 매니페스트는 행 추가만 → 병합
 → CI가 증거 보관본 업로드와 "보관본" 행 추가
```

- **테스트 먼저**: 로직(`jzip-shared`, `jzip-core`)은 반드시 단위 테스트를 먼저 쓴다. 화면(`JZip.exe`)은 PresentationModel 단위 테스트 + 화면 증거(E5)로 대신한다.
- **테스트 기대값은 명세에서**: 기대값을 구현 결과에서 베끼지 않는다. 명세의 수용 기준이나 코퍼스의 정답 파일에서 가져온다.

---

## 7. 브랜치·커밋·PR

| 항목 | 규칙 |
|---|---|
| 브랜치 | `<종류>/<작업 ID>-<짧은 이름>` 예) `feat/G2-F17-T03-encoding-score` |
| 커밋 | Conventional Commits + 작업 ID. 예) `feat(shared): score encoding candidates [G2-F17-T03]`. 증거 매니페스트는 코드 커밋과 섞지 않고 `work/evidence/**`만 바꾸는 커밋으로 따로 둔다 |
| PR | 작업 하나당 하나. 템플릿: 근거(FR/AC), 변경 요약, **증거**, 남긴 임시 코드(있으면 이유), 미결 사항 |
| CI 경로 필터 | `work/evidence/**`만 바꾼 커밋은 다시 빌드하지 않는다. 워크플로 단위 `paths-ignore`는 쓰지 않는다(필수 검사가 대기 상태로 남아 병합이 막힘). 대신 첫 작업(경로 필터 작업)이 바뀐 경로를 보고, 증거 전용이면 빌드·테스트 작업을 **작업(job) 단위로** 건너뛰며 필수 검사는 성공으로 보고한다. CI는 매니페스트에 적힌 코드 커밋 SHA가 "PR 머리에서 매니페스트 전용 커밋을 뺀 마지막 커밋"과 같은지 검사한다 |
| 병합 | squash 병합. 병합 뒤에도 매니페스트의 코드 커밋 SHA는 `refs/pull/<번호>/head`로 도달할 수 있고, CI 봇이 "보관본" 행에 `main`의 squash 병합 커밋 SHA를 함께 적는다([10](10-logging-evidence-operations.md) 3.3). `main`에 직접 push와 강제 push 금지. 예외는 CI 봇의 증거 보관본 커밋 하나뿐이며 `work/evidence/**`만 바꿀 수 있다 |
| 금지 | 테스트 비활성화(`DISABLED_`, `GTEST_SKIP` 남용), 매니페스트가 참조하는 증거의 수정·삭제, 서명 인증서·비밀 커밋, 작업 범위 밖 수정 |
| 허용 | 어떤 매니페스트도 참조하지 않는 로컬 진단 파일(임시 `.etl`, 덤프, 캡처 원본)은 지워도 된다. 매니페스트가 참조하는 항목은 업로드(CI 아티팩트·릴리스 자산)가 끝난 뒤에만 로컬에서 지운다 |

---

## 8. 완료 정의 (Definition of Done)

작업은 아래를 모두 만족해야 완료다.

- [ ] 작업 카드의 근거 FR과 수용 기준(태그에 맞는 범위)을 모두 충족. FR이 없는 카드는 분류별 기준을 충족: `G0-INFRA` = 3.2 게이트 항목(번호), `CLEANUP` = [10](10-logging-evidence-operations.md) 5절 표의 행, `UPSTREAM` = 10절 단계, `OPS` = OPS 명세의 FR·수용 기준(4절 표)
- [ ] 새 로직에 단위 테스트, 관련 코퍼스 통합 테스트 통과
- [ ] CI 3종(VS 2026 x64, VS 2022 x64, ARM64) 통과. ARM64 러너를 쓸 수 없으면 x64 러너의 ARM64 교차 컴파일 성공으로 대신하고 PR에 그 사실을 적는다([07](07-tech-stack.md) 8절)
- [ ] 증거 매니페스트(`work/evidence/<작업 ID>/EVIDENCE.md`)를 스크립트로 생성, 코드 커밋 SHA가 CI 검사와 일치, 필요한 증거 종류 모두 포함. verifier의 `JZIP-VERIFY` 코멘트 뒤에는 행 추가만([10](10-logging-evidence-operations.md) 3.3)
- [ ] 게이트 사전 검사 통과(임시 코드 표시 규칙 준수, [10](10-logging-evidence-operations.md) 4절)
- [ ] reviewer 승인, verifier 재현 확인(PR 코멘트 E9)
- [ ] 명세와 다르게 만든 부분이 있으면 명세 수정 PR 또는 ADR

---

## 9. 결정 기록 (ADR)

`work/decisions/ADR-<번호>-<제목>.md`. 시제품 결론, 명세 변경, 의존성 추가·버전 변경, 원본 패치 결정은 모두 ADR로 남긴다.

```markdown
# ADR-0007 셸 확장 설정 저장 위치
- 상태: 제안 / 채택 / 폐기
- 날짜:
- 배경:
- 결정:
- 근거(증거 링크):
- 영향받는 문서:
```

---

## 10. 7-Zip 업스트림 갱신

1. 별도 작업(`G<n>-UPSTREAM-T<번호>`)으로 `git subtree pull --prefix upstream/7zip` — 다른 작업과 섞지 않는다
2. `upstream/PATCHES.md`의 패치를 하나씩 확인하고 충돌 해결
3. 코퍼스 통합 테스트 전체 실행, 결과를 이전 버전과 비교한 표를 증거로 남김
4. [research/7zip-26.04-source-facts.md](research/7zip-26.04-source-facts.md)처럼 새 버전 기준 근거 자료를 갱신

---

## 11. 도구 계약

모든 도구는 PowerShell 스크립트이며 종료 코드는 **0 = 성공, 1 = 검사·실행 실패, 2 = 사용법 오류(매개변수 누락·잘못된 값)** 로 통일한다. 실행한 명령과 구성·정의는 `out/runlog.jsonl`에 한 줄씩 남기고, `new-manifest.ps1`이 이를 읽어 매니페스트의 "실행한 명령" 표를 만든다.

| 도구 | 매개변수 | 출력 | 만드는 작업 |
|---|---|---|---|
| `tools/build.ps1` | `-Config Debug\|Dev\|Test\|Release`, `-Arch x64\|arm64`, `[-Target all\|shared\|win\|core\|engine\|app\|shell\|package]` | `out/<Config>/<Arch>/` 바이너리, MSBuild `/bl` 로그, nmake 출력 로그(E1). `shared`·`win`은 `-mt`·`-md` 두 변형을 만든다. nmake는 구성에 맞는 정의(`JZIP_DEV_TRACE=1` 등)와 `JZIP_SHARED_LIB`·`JZIP_WIN_LIB`(`-mt` 변형)로 호출 | G0-INFRA-T02 |
| `tests/unit/run.ps1` | `-Config`, `-Arch`, `[-Filter <GoogleTest 필터>]` | `out/test/unit-<이름>.xml`(JUnit XML, E2) | G0-INFRA-T04 |
| `tests/integration/run.ps1` | `-Config Release\|Test`, `-Arch`, `[-Suite <이름>]`, `[-Corpus <경로>]` | `out/test/corpus-<이름>.csv`·`.md`(아카이브별 기대값·실제값·결과, E3). E3 증거로 쓰려면 `Release`, 테스트 훅에 의존하는 묶음은 `Test` | G0-INFRA-T05 |
| `tests/corpus/generate.ps1` | `[-Set v0\|full]`, `-Out <경로>` | 생성한 아카이브 + `manifest.csv`(파일, SHA-256, 정답 파일) | G0-INFRA-T05 |
| `tests/perf/run.ps1` | `-Config Release`, `-Arch`, `-Scenario <이름>` | `out/perf/<이름>.json`(수치 + CPU·OS 빌드·아키텍처, E8) | G0-INFRA-T09 (뼈대), 하네스는 G1 시제품에서 승격 |
| `tools/evidence/new-manifest.ps1` | `-Task <작업 ID>`, `[-RunLog <경로>]`, `[-CiRun <URL>]` | `work/evidence/<작업 ID>/EVIDENCE.md`(코드 커밋 SHA, 환경, 실행 명령, 산출물 해시) | G0-INFRA-T07 |
| `tools/evidence/trace-start.ps1` | `-Session <이름>`, `[-Keywords Ops\|Dev\|All]`, `[-Out <폴더>]` | `JZip` 공급자 ETW 세션 시작 | G0-INFRA-T06 |
| `tools/evidence/trace-stop.ps1` | `-Session <이름>` | `.etl` + `tracefmt` 변환 텍스트(E4) | G0-INFRA-T06 |
| `tools/evidence/capture-window.ps1` | `-Process JZip\|explorer`, `-Element <요소 ID>`, `-Theme light\|dark`, `-Scale 100\|150\|200`, `-Task <작업 ID>` | 축소 PNG(200KB 이하) `work/evidence/<작업 ID>/screens/<요소>-<테마>-<배율>.png`(E5). 대화형 세션이 아니면 1로 종료 | G0-INFRA-T08 |
| `tools/gate/check-task.ps1` | `-Task <작업 ID>`, `[-Base <git ref>]` | 검사 보고서 `out/gate/<작업 ID>.md`([10](10-logging-evidence-operations.md) 4절 항목) | G0-INFRA-T10 |
| `tools/gate/check-stage.ps1` | `-Stage G<n>` | 단계 보고서 `out/gate/stage-G<n>.md`([10](10-logging-evidence-operations.md) 5절 항목). DoD 검사에서 `G<n>-CLEANUP-T01` 카드는 제외 | G0-INFRA-T10 |
