# 08. 아키텍처·디자인 패턴·확장성

[07. 기술 스택](07-tech-stack.md)의 구성 요소(`JZip.exe`, `jzip-core.lib`, `jzip-shared.lib`, `jzip-win.lib`, `7z.dll`, `jzip-shell.dll`, `7z.exe`)를 어떤 구조와 패턴으로 만들지 정한다.
목표는 세 가지다.

1. **로직은 UI 없이 테스트할 수 있어야 한다** — 코딩 에이전트가 증거를 자동으로 만들 수 있게.
2. **새 기능(포맷, 메뉴 명령, 화면, 설정, 언어)을 기존 코드 수정 없이 추가할 수 있어야 한다.**
3. **최신 기술에 묶이는 범위를 최소화한다** — WinUI 3·C++/WinRT 의존은 화면 계층에만 둔다.

## 1. 계층 구조

```
┌──────────────────────────────── 표현 계층 ─────────────────────────────────┐
│ JZip.exe (WinUI 3)               jzip-shell.dll (탐색기 안)   7z.exe (CLI)  │
│  View(XAML) ⇄ ViewModel           IExplorerCommand            UI/Console    │
│  (C++/WinRT, WinRT 구현 주입)                                                │
└───────┬───────────────────────────────┬──────────────────────────┬─────────┘
        │ C++ 인터페이스 (WinRT 타입 금지)│ 실행 인자 프로토콜 (5절)  │
┌───────▼──────────── 응용 계층 (jzip-core) ─────────┐               │
│ 작업 서비스: Extract / Compress / Test / Browse /   │               │
│ Repair · OperationRunner · 설정 서비스 · 현지화      │               │
│ 서비스 · 암호 보관(ISecretStore·IUserConsent 포트)   │               │
└───────┬────────────────────────────────────────────┘               │
┌───────▼──────────── 기반 계층 (jzip-core) ─────────┐               │
│ 어댑터: 7-Zip 엔진(UI/Common·UI/Agent 래핑),         │               │
│ ZipRegistry 대체 설정 어댑터, 앱 전용 Win32 어댑터    │               │
│ (DPAPI, AMSI, WIC), 파일 로그                        │               │
└───────┬────────────────────────────────────────────┘               │
        │ (jzip-shell은 jzip-core를 링크하지 않음)                       │
┌───────▼──────────── 플랫폼 계층 (jzip-win, Win32 API만) ◀──────────────┤
│ jzip-shared 포트의 Win32 구현: 코드페이지 변환기(#17) · 파일 시스템 확인(#13) · │
│ 휴지통 IFileOperation(#31) · 설정·Lang 파일 읽기(공유 읽기, 마지막 캐시)      │
│ 표준 C++20 + Win32 API — WinRT·XAML·Windows App SDK 없음                     │
└───────┬────────────────────────────────────────────────────────────┘    │
        │ 포트 구현                                                       │
┌───────▼──────────── 도메인 계층 (jzip-shared, 순수 라이브러리) ◀───────┘
│ 인코딩 후보 점수(#17) · 스마트 해제 계획(#13) · tar 한 번에 판정(#12) ·     │
│ 분할 세트 규칙(#33) · 화면 상태 계산(PresentationModel) · 설정 스키마 +    │
│ JSON 읽기/쓰기 · 명령 등록부(메뉴 모델 #18 #34) · Lang 파서(#22, CLang 이식) │
│ 표준 C++20만 — OS 호출 없음(Win32·WinRT 없음), I/O·OS 기능은 포트로만      │
└──────────────▲─────────────────────────────────────────────────────────┘
               │ JZIP-PATCH 훅 지점에서만 호출 (jzip-shared 판단 + jzip-win 포트 구현)
          7z.dll (포맷·코덱, IInArchive / IOutArchive) ◀── jzip-core가 로드
```

링크 관계:

| 바이너리 | 링크하는 라이브러리 | 비고 |
|---|---|---|
| `JZip.exe` | `jzip-core.lib`, `jzip-shared-md.lib`, `jzip-win-md.lib` | WinRT 구현(알림, Windows Hello, `ApplicationData`)을 코어 포트에 주입 |
| `jzip-core.lib` | (정적 라이브러리) `jzip-shared`·`jzip-win`의 헤더와 포트를 사용 | 최종 링크는 이를 링크하는 바이너리(`JZip.exe`, 테스트)에서 `-md` 변형으로 |
| `jzip-shell.dll` | `jzip-shared-md.lib`, `jzip-win-md.lib`만 | `jzip-core`와 WinRT를 링크하지 않는다. CI가 링크 입력을 검사 |
| `7z.dll`, `7z.exe` | `jzip-shared-mt.lib`, `jzip-win-mt.lib` | nmake 빌드. makefile 변수(`JZIP_SHARED_LIB`, `JZIP_WIN_LIB`)가 MSBuild로 만든 `-mt` 변형을 가리킨다([07](07-tech-stack.md) 5절 플래그 계약, G0 산출물) |
| 테스트 | `jzip-shared-md.lib`, `jzip-win-md.lib`, `jzip-core.lib` | 가짜 포트 사용 |

의존 방향 규칙:

| 규칙 | 이유 |
|---|---|
| 의존은 위에서 아래로만. `jzip-shared`는 표준 C++20 외에 아무것도 참조하지 않는다(OS 호출 없음 — Win32·WinRT 없음, 파일·레지스트리·네트워크 I/O와 OS 기능은 주입된 포트로만) | 도메인 로직을 빠르고 결정적으로 테스트. 7z.dll·7z.exe·셸·앱이 같은 판단을 공유 |
| `jzip-win`은 Win32 API와 표준 C++20만 쓴다(WinRT·XAML·Windows App SDK 없음). `7z.dll`·`7z.exe`·셸이 필요로 하는 `jzip-shared` 포트(코드페이지 변환기, 파일 시스템 확인, 휴지통, 설정·Lang 파일 읽기)의 Win32 구현만 둔다. 앱에서만 쓰는 어댑터(DPAPI, AMSI, WIC)는 `jzip-core`에 둔다 | 업스트림 바이너리와 셸이 `jzip-core` 없이 같은 Win32 구현을 공유 |
| `jzip-core`는 WinRT·XAML·Windows App SDK를 참조하지 않는다. 설정 폴더·패키지 폴더 경로는 `JZip.exe`가 구해 **경로로 주입**한다 | 화면 기술이 바뀌어도 로직은 그대로 |
| WinRT 전용 기능은 코어에 포트만 두고 `JZip.exe`가 구현을 주입한다: 알림 `INotifier`(`AppNotificationManager`), Windows Hello `IUserConsent`(`UserConsentVerifier`). 암호 저장은 `ISecretStore`(DPAPI 구현은 코어 기반 계층) | 코어를 앱 프레임워크에서 분리 |
| `jzip-shell.dll`은 `jzip-shared`·`jzip-win`만 링크하고 아카이브를 열지 않는다. 확장자·설정만 보고 메뉴를 만들고, 실제 작업은 `JZip.exe`에 넘긴다. CI가 셸 프로젝트의 링크 입력에 `jzip-core.lib`와 WinRT 라이브러리(Windows App SDK·C++/WinRT)가 없는지 검사한다 | 탐색기 프로세스 안에서 느리거나 죽는 일을 막음. 셸의 의존 범위 고정 |
| 7-Zip 원본 파일은 `JZIP-PATCH` 훅 지점에서만 `jzip-shared`(판단)와 `jzip-win`(포트 구현)을 호출한다. 예: `Archive/Zip/ZipItem.cpp` `GetUnicodeString` → 인코딩 점수기(`jzip-shared`) + 코드페이지 변환기(`jzip-win`)(#17), `UI/Common/Extract.cpp` `DecompressArchive` → 스마트 해제 계획(`jzip-shared`) + 파일 시스템 확인(`jzip-win`)(#13), `UI/Common/Extract.cpp` `Extract()` → 휴지통(`jzip-win`)(#31), `UI/Common/OpenArchive.cpp`·압축 핸들러 → tar 한 번에 판정(`jzip-shared`)(#12) | 업스트림 변경의 영향 범위를 훅 지점으로 한정 |
| `jzip-core`는 7-Zip 코드를 기반 계층의 엔진 어댑터를 통해서만 호출한다 | 위와 같음 |
| `7z.exe`는 `settings.json`을 읽지 않는다. CLI 동작은 스위치(없으면 스키마 기본값)로만 정해진다. 설정 어댑터는 `jzip-core` 빌드에만 넣고, `7z.exe`는 `jzip-win`의 설정 파일 읽기도 호출하지 않는다 | 같은 명령이 PC 설정에 따라 다르게 동작하지 않게 함 |

---

## 2. 디자인 패턴

| 패턴 | 적용 위치 | 내용 |
|---|---|---|
| **MVVM** | `JZip.exe` | XAML View는 `x:Bind`로 ViewModel에만 연결한다. ViewModel은 얇게 두고, 화면 상태 계산(버튼 활성, 미리보기 문구 등)은 `jzip-shared`의 PresentationModel(순수 C++)에 둬서 UI 없이 테스트한다 |
| **포트와 어댑터** | 응용 ↔ 기반 계층, `jzip-shared` ↔ 호출자 | `IArchiveEngine`, `IFileSystem`, `IRecycleBin`, `ISecretStore`, `IUserConsent`, `INotifier`, `IClock`, `ILogger`, 코드페이지 변환기 인터페이스. Win32 구현은 `7z.dll`·`7z.exe`·셸이 쓰는 것은 `jzip-win`, 앱 전용은 `jzip-core` 기반 계층, WinRT 구현은 `JZip.exe`에 둔다. 테스트에서는 가짜 구현으로 바꾼다 |
| **명령(Command) + 작업 실행기** | 압축·해제·테스트·복구 | 작업 하나 = `Operation` 객체(입력, 진행, 취소, 결과). `OperationRunner`가 대기열(S4-07), 일시정지, 취소, 진행 보고를 공통으로 처리 |
| **전략(Strategy)** | 정책이 바뀌는 곳 | 폴더 만들기(스마트/항상/그대로, #13), 같은 파일 처리(묻기/덮어쓰기/건너뛰기/이름 바꾸기), 인코딩 후보 점수(#17), 해제 후 처리(#30·#31) |
| **관찰자(Observer)** | 진행 상황 | `IProgressSink`로 진행·경고·오류를 보낸다. ViewModel은 받은 이벤트를 UI 스레드로 옮겨 표시 |
| **등록부(Registry)** | 확장 지점 | 명령 등록부(4절), 포맷 정보 표, 설정 스키마, 화면 경로 표. 새 항목은 등록만 하면 동작 |
| **결과 값(Result)** | 계층 경계 | 예외는 모듈 경계(COM·ABI·스레드)를 넘기지 않는다. `Result<T, Error>`나 `HRESULT`로 돌려주고, `Error`는 06 문구 원칙에 맞는 사용자 메시지 ID를 가진다 |
| **설정 스키마 + 마이그레이션** | `jzip-shared` 스키마, `jzip-core` 설정 서비스 | 키마다 타입·기본값·버전·`pii` 표시. 버전이 바뀌면 마이그레이션 함수를 순서대로 적용 |

스레드 규칙:

- UI 스레드는 화면만 다룬다. 압축·해제·목록 읽기·썸네일은 작업 스레드에서 한다.
- 모든 장시간 작업은 취소 토큰을 받는다. 취소는 1초 안에 반응해야 한다.
- 코어는 스레드 안전한 객체만 공개한다. 공유 상태는 작업 실행기 안에만 둔다.

---

## 3. 모듈 구성 (`src/shared`, `src/shared-win`, `src/core`)

`src/shared` — `jzip-shared.lib` (도메인, 순수 라이브러리):

| 폴더 | 내용 | 관련 기능 |
|---|---|---|
| `encoding/` | 인코딩 후보 점수, 혼합 이름 처리. 바이트 → 문자 변환은 주입된 변환기로 | #17 #16 |
| `extract/` | 스마트 해제 계획, 경로 계산, 충돌 정책 | #13 #35 #30 |
| `tar/` | tar 한 번에 풀기 판정(안쪽이 tar인 단일 스트림인지) | #12 |
| `volume/` | 분할 세트 규칙(조각 이름 판정, 첫 조각·누락 계산) | #33 |
| `presentation/` | 화면 상태 계산(PresentationModel) | 06 전체 |
| `settings/` | 설정 스키마(키·타입·기본값·버전·`pii`), JSON 읽기/쓰기, 마이그레이션 함수 | S8, OPS |
| `commands/` | 명령 등록부(메뉴 모델: 명령 ID, 문자열 ID, 표시 조건, 기본 위치, 실행 인자) | #18 #19 #34 |
| `i18n/` | 7-Zip `CPP/Common/Lang.cpp` `CLang` 파싱 이식(`Lang\*.txt` 형식 검사·ID 조회), 언어 코드 → 파일 이름 표(`kLangs` 이식), 내장 영어 최소 문구표 | #22 |
| `ports/` | `ILogger`, 코드페이지 변환기, 파일 시스템 확인, 휴지통, 파일 읽기 등 `jzip-shared`가 쓰는 주입 인터페이스 | 전체 |

`src/shared-win` — `jzip-win.lib` (플랫폼, Win32 API만 — WinRT·XAML 없음, 표준 C++20):

| 폴더 | 내용 | 관련 기능 |
|---|---|---|
| `locale/` | 코드페이지 변환기 Win32 구현(`MultiByteToWideChar` + `MB_ERR_INVALID_CHARS`), `CP_OEMCP`/`CP_ACP` 실제 값 조회, Windows 표시 언어 조회("시스템 설정 따르기") | #17 #22 |
| `fs/` | 파일 시스템 확인: 대상 폴더·항목의 존재와 속성 | #13 #31 |
| `recycle/` | 휴지통: `IFileOperation` + `FOFX_RECYCLEONDELETE` | #31 |
| `files/` | 파일 읽기: 공유 읽기(`FILE_SHARE_READ`·`FILE_SHARE_WRITE`·`FILE_SHARE_DELETE`)로 열기, 마지막으로 읽은 내용 캐시(수정 시각·크기 비교). `settings.json`(셸)과 `Lang\*.txt`(셸·현지화 서비스)를 읽는다. 셸 설정 경로 결정(6절) | S8 #22 #34 |

`src/core` — `jzip-core.lib` (응용·기반):

| 폴더 | 계층 | 내용 | 관련 기능 |
|---|---|---|---|
| `app/operations/` | 응용 | Operation, OperationRunner, 작업별 서비스 | #20 #24 #30–#32 |
| `app/settings/` | 응용 | 설정 서비스(주입된 설정 폴더의 `settings.json` 읽기·병합·원자적 쓰기, 6절), ZipRegistry 대체 어댑터 | S8 |
| `app/i18n/` | 응용 | 현지화 서비스만(언어 선택, 파일 로드, `Get(id)`, 변경 알림). 파일 로드는 `jzip-win` 파일 읽기, 파싱은 `jzip-shared`(`CLang` 이식) | #22 |
| `app/secrets/` | 응용 | 암호 보관 서비스, `ISecretStore`·`IUserConsent` 포트 | #23 |
| `infra/engine/` | 기반 | 7-Zip 엔진 어댑터 | 전체 |
| `infra/win/` | 기반 | 앱 전용 Win32 어댑터: DPAPI, AMSI, WIC. `7z.dll`·`7z.exe`·셸도 쓰는 Win32 구현(코드페이지 변환기, 파일 시스템 확인, 휴지통, 파일 읽기)은 `jzip-win`에 있다 | #21 #23 #26 |
| `infra/log/` | 기반 | `ILogger` 파일 로그 구현([10](10-logging-evidence-operations.md)) | 전체 |

---

## 4. 확장성

새 기능은 아래 확장 지점에 **등록**하는 방식으로 추가한다. 등록 지점 밖의 기존 코드를 고쳐야 한다면 설계 검토(ADR) 대상이다.

| 추가할 것 | 확장 지점 | 해야 할 일 |
|---|---|---|
| 새 아카이브 포맷 | `7z.dll` 포맷 핸들러(7-Zip `REGISTER_ARC` 방식) + **포맷 정보 표** `src/engine/formats.def` | 핸들러 추가 → 표에 확장자·표시 이름·아이콘 등록. MSIX `FileTypeAssociation`, S8-07 목록, S2-08 같은 조건 목록은 이 표에서 **빌드 때 생성**한다. CI가 `formats.def`와 빌드한 `7z i` 출력(포맷·확장자 목록)을 비교해 어긋나면 실패 |
| 새 우클릭 명령 | 명령 등록부 `src/shared/commands` | 명령 ID, 문자열 ID, 표시 조건(선택 개수·확장자), 기본 위치(맨 위/하위/숨김), 실행 인자(5절 `--op`). S7 메뉴·S8-05 표·S1-02 `▾` 메뉴가 같은 등록부를 읽는다 |
| 새 작업 종류 | `Operation` 하위 클래스 | 입력·실행·결과 정의. 대기열·진행창·알림은 자동 적용 |
| 새 화면 | 화면 경로 표 + View + ViewModel + PresentationModel | 06에 요소 ID 추가 → 경로 등록 |
| 새 설정 | 설정 스키마 `src/shared/settings` | 키·타입·기본값·소속 탭·`pii` 여부. 키 이름은 점으로 구분한 PascalCase `Area.Key`(예: `Encoding.AutoDetect`, `Extract.FolderMode`). 기존 키를 바꾸면 마이그레이션 추가 |
| 새 언어 | `Lang\<언어>.txt` (7-Zip 형식) + `Lang\jzip\<언어>.txt` | 번역 파일 추가(코드 수정 없음). 매니페스트 표시 문구(앱 이름·설명·파일 형식 이름)는 `.resw`에도 그 언어 문자열을 추가해야 한다([F22](05-feature-specs/F22-languages.md) 7절) |
| 실험 기능 | 기능 플래그 | 설정 `Experimental.<Name>` (기본 꺼짐, 예: `Experimental.TabDragOut`). 정식 기능이 되면 플래그 제거 |

하지 않는 것:

- **외부 플러그인 DLL 로딩**: 7-Zip은 `Codecs`·`Formats` 폴더의 DLL을 불러올 수 있지만, j-zip은 MSIX 안의 서명된 구성만 쓴다. 보안과 증거 추적을 위해서다.

---

## 5. 셸 확장 ↔ 앱 연결 규약

**이 절이 규범이다.** 05 명세의 구현 지점은 이 형식을 따른다. `jzip-shell.dll`은 `JZip.exe`를 실행 인자로 호출하며, 규약은 버전을 가진다.

```
JZip.exe --proto 1 --op extract --mode smart --files-from <임시 목록 파일>
JZip.exe --proto 1 --op compress --format zip --files-from <임시 목록 파일>
JZip.exe --proto 1 --op open <아카이브 경로>
```

`--op` 목록 (proto 1):

| `--op` | 추가 인자 | 쓰는 메뉴 항목 |
|---|---|---|
| `extract` | `--mode smart` / `always` / `none` 또는 `--dialog`(S2 열기), `--files-from` | S7-01(`smart`), S7-02(`always`), S7-04(`--dialog`), S7-10(`smart`, 여러 대상) |
| `compress` | `--format zip` / `7z` 또는 `--dialog`(S3 열기), `--files-from` | S7-11(`--format`), S7-08·"하나로 압축"(`--dialog`) |
| `test` | `--files-from` | S7-06 |
| `open` | 아카이브 경로 | S7-05 |
| `recompress` | `--format 7z`, `--files-from` | S7-07 |
| `hash` | `--method <이름>`, `--files-from` | S7-13 |
| `settings` | `--page shell-menu` | S7-09 |

| 규칙 | 내용 |
|---|---|
| 버전 | `--proto` 값이 다르면 앱은 오류 안내 후 종료(설치 불일치 진단용) |
| 인자 범위 | 실행 인자는 메뉴 항목이 정하는 값(op, mode, format, 대상)만 담는다. S8 저장값(#12 S8-11, #30, #31, #17 자동 감지 등)은 `JZip.exe`가 `settings.json`에서 직접 읽는다 |
| 임시 목록 파일 | 셸이 자기 프로세스의 `%TEMP%`(`GetTempPathW`)에 고유 이름 `jzip-list-<임의값>.txt`(UTF-16, 한 줄에 경로 하나)로 만든다. 기본 사용자 ACL을 그대로 쓴다. 앱은 읽은 뒤 삭제한다. 셸은 다음 호출 때 자기가 만든 목록 파일 중 1일이 지난 것을 지운다 |
| `%TEMP%` 가상화 | 셸이 패키지 ID를 가진 채 실행되면 `%TEMP%`가 패키지 전용 경로로 가상화될 수 있다. 앱이 그 파일을 읽을 수 있는지 **SP-3**에서 확인하고 ADR로 기록한다 |
| 단일 창 | S8-20 값에 따른다. 켜져 있으면 이미 실행 중인 창의 새 탭·작업 대기열로 넘기고(#36), 꺼져 있으면 새 창을 연다(시제품 SP-3) |
| 검증 | 앱은 받은 인자를 그대로 신뢰하지 않고 경로 존재·형식을 다시 확인한다 |

---

## 6. 설정 저장 위치 (앱 ↔ 셸 확장 공유)

앱(`JZip.exe`)과 셸 확장(`jzip-shell.dll`)은 같은 설정(우클릭 메뉴 구성 #34, 언어 #22 등)을 읽어야 한다. 근거: [research/ops-research-2026-10.md](research/ops-research-2026-10.md) Q1–Q3.

**결정: 패키지 LocalState 폴더의 JSON 파일 하나. 7-Zip 레지스트리 저장은 쓰지 않는다.**

```
%LOCALAPPDATA%\Packages\<패키지 패밀리 이름>\LocalState\settings.json
```

| 항목 | 규칙 |
|---|---|
| 7-Zip 설정 코드 | 7-Zip `UI/Common/ZipRegistry.cpp`의 레지스트리 읽기·쓰기를 `JZIP-PATCH`로 설정 어댑터 호출로 바꾼다. 기존 구조체(`NExtract::CInfo`, `CContextMenuInfo` 등)는 그대로 쓰고 값의 저장소만 `settings.json`이다. 구조체 필드와 스키마 키의 대응은 스키마에 둔다. 확인: 7-Zip 26.04에서 `ZipRegistry.cpp`를 컴파일하는 makefile은 `UI/FileManager`, `UI/GUI`, `UI/Explorer`, `UI/Far`, `Bundles/Fm`뿐이고 `UI/Console`(7z.exe)은 포함하지 않으므로 7z.exe에는 어댑터가 필요 없다 |
| 7-Zip 설정 가져오기 | 처음 실행할 때 한 번 7-Zip 레지스트리 값을 **읽기 전용**으로 가져온다(F34 FR-34-6). MSIX 앱의 HKCU 읽기는 실제 HKCU와 병합되어 보인다(research Q2). j-zip은 레지스트리에 쓰지 않는다 |
| 경로 결정 (앱) | `JZip.exe`가 `ApplicationData::Current().LocalFolder()`로 설정 폴더를, `GetCurrentPackagePath`로 패키지 설치 폴더(`Lang\` 등)를 구해 `jzip-core`에 경로로 주입한다. `jzip-core`는 WinRT를 호출하지 않는다 |
| 쓰기 | 쓰는 쪽(`JZip.exe`, 여러 인스턴스 가능)은 ① 명명된 뮤텍스 `Local\JZip.Settings`를 잡고 ② 파일을 다시 읽어 바꾼 키만 병합하고 ③ 같은 폴더에 고유 임시 파일(`settings.json.<pid>-<임의값>.tmp`)로 쓴 뒤 ④ `ReplaceFileW`(대상이 없으면 `MoveFileExW(MOVEFILE_REPLACE_EXISTING)`)로 교체한다. 교체가 공유 위반으로 실패하면 최대 5회, 100ms 간격으로 재시도하고, 그래도 실패하면 임시 파일을 지우고 설정 서비스에 `Result` 오류(사용자 메시지 ID를 가진 `Error`, 2절 결과 값 규칙)를 돌려준다. S8은 그 메시지 ID의 오류 문구를 표시하고, 실패는 운영 로그에도 남긴다 |
| 읽기 (앱) | 설정 서비스가 시작 시 읽고, 바뀌면 위 쓰기 규칙으로 저장 |
| 읽기 (셸 확장) | `jzip-win` 파일 읽기(3절 `files/`)로 `FILE_SHARE_READ`·`FILE_SHARE_WRITE`·`FILE_SHARE_DELETE`를 모두 지정해 열어 앱의 교체를 막지 않는다. 파싱에 실패하면 **마지막으로 읽은 캐시 값**을 계속 쓰고, 한 번도 읽지 못했을 때만 스키마 기본값을 쓴다 |
| 셸 확장 경로 | 패키지 패밀리 이름(PFN)을 아래 순서로 구해 `FOLDERID_LocalAppData\Packages\<PFN>\LocalState\settings.json`을 읽는다. 고정 PFN은 쓰지 않는다. ① DLL이 패키지 ID를 가진 채 실행되면 `GetCurrentPackageFamilyName` → ② 실패하면 **DLL 자기 경로에서 유도**: `GetModuleFileNameW(hModule)` → 설치 폴더 이름(패키지 전체 이름) → `PackageFamilyNameFromFullName` → ③ 둘 다 실패하면 스키마 기본값을 쓰고 ETW 이벤트 `Shell.SettingsPath`를 `source=none`으로 남긴다. ①②에서도 어느 단계를 썼는지(`source`)를 같은 이벤트로 남긴다([10](10-logging-evidence-operations.md) 2.2). Visual Studio 느슨한 레이아웃 배포(F5 실행)도 패키지 ID를 가지므로 ①을 탄다 — SP-1에서 확인한다 |
| 셸 확장 성능 | 파일 수정 시각·크기가 같으면 캐시를 쓴다. 메뉴 표시 함수(`GetState`·`GetTitle`)에서는 무거운 I/O를 하지 않는다 |
| 형식 | `{"schemaVersion": 1, ...}` — 설정 스키마(2절)의 버전·마이그레이션 규칙을 따른다. 키는 `Area.Key`(4절) |
| 민감 키 | 최근 경로, 마지막 풀 위치처럼 경로를 담는 키는 스키마에 `pii: true`로 표시한다. 로그 내보내기는 "경로 포함"이 꺼져 있으면 이 키를 뺀다([OPS 진단](05-feature-specs/OPS-diagnostics.md)) |
| 암호 저장 파일 | #23 저장 파일 `LocalState\passwords.json`도 같은 쓰기·읽기 규칙을 따른다(뮤텍스 이름 `Local\JZip.Passwords`) |
| CLI | `7z.exe`는 이 파일을 읽지 않는다(1절) |

채택하지 않은 방식:

| 방식 | 이유 |
|---|---|
| HKCU 레지스트리 (7-Zip `ZipRegistry` 그대로) | MSIX 앱의 HKCU 쓰기는 앱 전용 가상 하이브로 들어가서 탐색기에서 보이지 않는다. 가상화를 끄려면 제한된 기능(`unvirtualizedResources`)이 필요하고, Microsoft는 이를 게임 등 특정 용도로만 안내해 Store 심사 위험이 있다 |
| `ApplicationData.LocalSettings` | 앱 전용 레지스트리 하이브에 저장되어 API로만 읽을 수 있다. 셸 확장이 패키지 ID 없이 실행되면 읽지 못한다 |
| 게시자 캐시 폴더 | 여러 패키지 간 공유용이고 패키지 ID가 필요하다 |

검증: 셸 확장이 실제로 어느 프로세스(`explorer.exe` 또는 대리 프로세스)에서, 패키지 ID를 가진 채 실행되는지는 문서로 확정되지 않았다. 시제품 **SP-1**에서 셸 확장이 `GetCurrentPackageFamilyName`·`GetModuleFileNameW` 결과, 유도한 PFN, 사용한 단계(①②③), 설정 파일 읽기 성공 여부를 로그로 남겨 확인하고 ADR로 기록한다. MSIX 설치와 Visual Studio 느슨한 레이아웃 배포를 모두 확인한다.

---

## 7. 의존성 검토 — 최신 환경에 너무 묶이지 않았나

근거: [research/tech-research-2026-10.md](research/tech-research-2026-10.md), [research/ops-research-2026-10.md](research/ops-research-2026-10.md) Q4–Q6.

### 7.1 무엇이 최신 기술에 의존하나

| 의존 대상 | 출시 시점 | 의존하는 구성 요소 | 영향 범위 |
|---|---|---|---|
| Windows App SDK 2.5.x | 2026-09 | `JZip.exe`만 | 화면 계층 |
| C++/WinRT 3.0 | 2026-05 (첫 3.0) | `JZip.exe`만 | 화면 계층 |
| Visual Studio 2026 | 2026 | 빌드 기본값 | 빌드 환경 |
| MSIX 패키지 기능(셸 등록, 파일 연결, 알림) | `desktop4:FileExplorerContextMenus`는 Windows 10 1803(17134)부터 제공(research Q1). 19041은 sparse package용 `AllowExternalContent`의 도입 버전이며 j-zip은 쓰지 않음. j-zip 최소 OS는 22H2 | 패키징 | 배포 |
| C++20 | — | 새 코드 전체 | VS 2022에서도 지원 |
| `jzip-shared`, `jzip-win`, `jzip-core`, `7z.dll`, `jzip-shell.dll`, `7z.exe` | — | 표준 C++ + Win32 API만 (`jzip-shared`는 표준 C++만) | **최신 SDK 의존 없음** |

판단: **최신 의존성은 화면 계층(`JZip.exe`)에 몰아 두었고, 로직·엔진·셸·CLI는 Win32와 표준 C++만 쓴다.** 그래서 Windows App SDK나 C++/WinRT에 큰 변경이 생겨도 영향은 화면 코드에 그친다. CLI(`7z.exe`)는 Windows App SDK 없이 빌드·테스트할 수 있다.

### 7.2 위험과 대응

| 위험 | 사실 | 대응 |
|---|---|---|
| Windows App SDK 주 버전 변경 | 2.0부터 SemVer, 주 버전은 6개월 이상 간격, 깨지는 변경은 주 버전에서만. 릴리스당 서비스 기간은 약 12개월(표에서 본 패턴이지 공식 정책은 아님) | 정확한 버전 고정(NuGet lock). 주 버전 업그레이드는 별도 작업 + ADR. 서비스 종료 3개월 전에 다음 버전 검토 |
| C++/WinRT 3.0이 새 버전 | 비표준 코루틴 제거 등 깨지는 변경 포함. CI는 VS 2022(v143)·VS 2026(v145) 둘 다 테스트. 최소 컴파일러 버전은 명시 없음 | 버전 고정. VS 2022 빌드를 CI에서 계속 유지해 특정 컴파일러에 묶이지 않게 함 |
| VS 2026 전용 기능에 의존 | — | 코드 규칙: VS 2022에서 컴파일되지 않는 기능 사용 금지(CI `windows-2022`가 검사) |
| 최소 OS | 2004는 2021-12에 서비스 종료되었고 Windows App SDK 공식 지원 OS 표에 없다. Windows 10 전체는 2025-10-14 지원 종료(소비자 ESU 2027-10-12까지) | **Windows 10 22H2(19045) 이상으로 결정**(2026-10-14 사용자 결정). Windows 10 ESU 종료 시점에 Windows 11 전용 전환을 ADR로 검토 |
| 오래된 OS를 못 쓰게 됨 | Windows 7·8.1은 지원 안 함 | 해당 사용자는 7-Zip 원본 사용 안내. j-zip 목표 범위 밖 |
| 의존성 공급망 | NuGet 패키지 3종(Microsoft), 동봉 라이브러리 2종(LZ4, unalz), vcpkg(테스트 도구) | NuGet 잠금 파일, vcpkg 매니페스트의 `builtin-baseline` 고정, 동봉 라이브러리는 버전·해시를 `third_party/VERSIONS.md`에 기록, 갱신은 ADR |

### 7.3 업그레이드 정책

| 대상 | 정책 |
|---|---|
| 7-Zip 업스트림 | 새 정식 버전이 나오면 다음 단계 시작 전에 반영([09](09-development-process.md) 10절) |
| Windows App SDK | 안정판만 사용. 부 버전(2.x)은 분기마다 검토, 주 버전은 ADR |
| C++/WinRT, WIL | [07](07-tech-stack.md) 7절의 버전을 NuGet에 **명시적으로 고정**한다(C++/WinRT 3.0.x). Windows App SDK 2.5 템플릿이 실제로 참조하는 C++/WinRT 버전은 G0에서 확인해 ADR로 남기고(research Q4: 공식 문서 [미확인]), 고정 버전 변경은 ADR |
| Visual Studio | 최신 2개 메이저(VS 2026, VS 2022)에서 빌드되도록 유지 |
| LZ4, unalz | 보안 수정이 있을 때만 갱신 |
