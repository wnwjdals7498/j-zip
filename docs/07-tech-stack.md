# 07. 기술 스택 결정

j-zip의 언어, UI 기술, 빌드, 배포, 의존성을 정한다. 조사 근거는 [research/tech-research-2026-10.md](research/tech-research-2026-10.md)(기준일 2026-10-14)와 [research/7zip-26.04-source-facts.md](research/7zip-26.04-source-facts.md)에 있다.

## 1. 결정 요약

| 영역 | 결정 | 상태 |
|---|---|---|
| 언어 | C++ — 7-Zip 코어는 기존 스타일 유지, 새 코드는 **C++20** | 확정 |
| UI | **WinUI 3** (Windows App SDK **2.5.x**) + **C++/WinRT 3.0** — 독립 WinUI 3 앱(XAML Islands 아님) | 확정 (사용자 결정) |
| 배포 | **MSIX** (서명된 사이드로드 + Microsoft Store), WinGet으로도 배포 | 확정 (사용자 기준: 우클릭 메뉴·기본 앱 등록) |
| 최소 OS | **Windows 10 2004 (빌드 19041)** 이상, x64 · ARM64 | 확정 (사용자 결정) |
| 컴파일러 | MSVC, Visual Studio 2026 기본 (VS 2022 호환 유지) | 확정 |
| 빌드 | 7-Zip 코어 = 기존 **nmake** / 새 프로젝트 = **MSBuild** 솔루션 | 확정 |
| 패키지 관리 | NuGet(Microsoft 패키지), vcpkg 매니페스트(테스트 도구), 소스 동봉(`third_party/`) | 확정 |
| 테스트 | GoogleTest(단위) + 샘플 아카이브 코퍼스 통합 테스트 | 확정 |
| CI | GitHub Actions — `windows-2025`(VS 2026), `windows-2022`, `windows-11-arm` | 확정 |
| 제외 | 포터블 배포, Mile.Xaml, Qt, WPF/.NET, WebView2 | 확정 |

---

## 2. 배경: 사용자 기준에서 나온 결론

사용자 기준은 **"탐색기 우클릭 메뉴에 등록되고, 기본 앱으로 등록해 쓸 수 있어야 한다"**이다.

- Win11 새 우클릭 메뉴(#19)는 앱에 **패키지 ID**가 있어야 등록된다. 방법은 MSIX 또는 sparse package(외부 위치 패키징)뿐이고, 둘 다 **코드 서명이 필수**다. 포터블은 설치 경로가 고정되지 않아 부적합하다.
- WinUI 3는 Windows App Runtime이 필요하다. 언패키지드로 배포하면 런타임 설치 프로그램과 Bootstrapper를 따로 챙겨야 하지만, **MSIX는 프레임워크 패키지 의존성으로 런타임을 자동 설치**한다.
- MSIX 매니페스트의 `uap:FileTypeAssociation`으로 지원 확장자를 등록하면 Windows "기본 앱" 후보가 된다. Windows 8 이후로는 앱이 기본 앱을 스스로 바꿀 수 없으므로, 사용자가 설정에서 고르도록 안내한다(S8-07 → `ms-settings:defaultapps`).
- 완료 알림(#32)의 `AppNotificationManager`도 패키지 앱에서 가장 단순하다.

→ WinUI 3 + MSIX가 세 요구(우클릭 메뉴, 기본 앱, 알림)를 한 번에 만족한다. sparse package + MSI는 같은 서명 비용에 설치 구성만 복잡해지므로 쓰지 않는다. **포터블 배포는 제외**한다.

---

## 3. 구성 요소와 아키텍처

```
j-zip.msix  (Win10 2004+, x64 / ARM64, 프레임워크 의존: Microsoft.WindowsAppRuntime 2.x)
├─ JZip.exe          WinUI 3 앱 — S1~S9, D1~D3 전부. C++/WinRT 3.0, C++20
│   └─ links ─▶ jzip-core.lib
├─ jzip-core.lib     7-Zip UI/Common + UI/Agent 로직 + j-zip 로직(스마트 해제, 인코딩 감지 등)
│   └─ loads ─▶ 7z.dll
├─ 7z.dll            포맷·코덱 엔진 (7-Zip + ALZ / EGG / LZ4 핸들러)
├─ jzip-shell.dll    IExplorerCommand 셸 확장 (UI/Explorer/ContextMenu.cpp 기반, XAML 없음)
│   └─ launches ─▶ JZip.exe --extract … / --compress … (현재 7zG 역할)
└─ 7z.exe            명령줄 (7-Zip 콘솔 그대로)
```

| 구성 요소 | 기반 (7-Zip 26.04) | 바뀌는 점 |
|---|---|---|
| `JZip.exe` | `UI/FileManager`(7zFM) + `UI/GUI`(7zG)를 **대체** | 화면은 WinUI 3로 새로 작성. 기존 로직은 `jzip-core.lib`로 옮겨 재사용 |
| `jzip-core.lib` | `UI/Common`, `UI/Agent` | 06·05 명세의 로직 변경(F13 스마트 해제, F17 인코딩, F12 tar 한 번에, F30–F32 등). UI와 무관한 순수 C++ |
| `7z.dll` | `Bundles/Format7zF` | ALZ/EGG/LZ4 핸들러 추가(F07, F08). nmake 빌드 유지 |
| `jzip-shell.dll` | `UI/Explorer` | `CZipContextMenu`(`IExplorerCommand` 구현 이미 있음)를 MSIX `windows.fileExplorerContextMenus`로 등록. S7 항목 구성 변경 |
| `7z.exe` | `UI/Console` | 거의 그대로. F17 등 코어 변경은 자동 반영 |

설계 원칙:

- **화면과 로직 분리**: `JZip.exe`는 화면과 상태만 담당하고, 압축·해제·목록 로직은 `jzip-core.lib`에 둔다. 그래서 CLI(`7z.exe`)와 GUI가 같은 동작을 보장하고, 로직은 UI 없이 단위 테스트할 수 있다.
- **7-Zip 코드는 최소 수정**: 업스트림 갱신을 따라가기 위해 7-Zip 소스는 `upstream/7zip`(git subtree)에 두고, 변경은 작은 패치로 유지한다. 새 기능은 가능한 한 새 파일로 추가한다.
- **WinUI 3 단일 스택**: Microsoft 문서가 시스템 XAML(Mile.Xaml 계열)과 WinUI 3의 혼용을 금지하므로 NanaZip의 Mile.Xaml UI 코드는 가져오지 않는다. NanaZip은 패키징·셸 등록 방식을 참고하는 용도로만 쓴다(로고·아이콘은 CC BY-ND라 사용 금지).

---

## 4. 언어와 코드 규칙

| 범위 | 표준 | 규칙 |
|---|---|---|
| 7-Zip 원본 파일 | 7-Zip 기존 방식 | STL·예외를 새로 들이지 않음. `CMyComPtr`, `UString` 등 7-Zip 타입 유지. 수정은 최소화 |
| `jzip-core` 새 파일 | C++20 | STL 사용 가능. 7-Zip 인터페이스(`IInArchive` 등) 경계에서는 7-Zip 타입 사용 |
| `JZip.exe` | C++20 + C++/WinRT 3.0 | 코루틴(`co_await`)으로 비동기 처리, 장시간 작업은 백그라운드 스레드에서. WIL로 Win32 핸들·HRESULT 관리 |
| `jzip-shell.dll` | C++20 | XAML·WinRT UI 금지(탐색기 프로세스 안에서 동작). 무거운 작업은 `JZip.exe`로 넘김 |

---

## 5. 빌드

| 대상 | 빌드 도구 | 비고 |
|---|---|---|
| `7z.dll`, `7z.exe` | nmake (`CPP/Build.mak`) | 업스트림 방식 그대로. MSBuild "Makefile 프로젝트"로 감싸 솔루션에서 함께 빌드 |
| `jzip-core.lib`, `JZip.exe`, `jzip-shell.dll`, 테스트 | MSBuild (`jzip.sln`, `.vcxproj`) | `JZip` 프로젝트는 WinUI 3 단일 프로젝트 MSIX(`WindowsPackageType=MSIX`) |
| 패키지 | MSBuild MSIX 패키징 → `j-zip_{ver}_{arch}.msix` + `.msixbundle` | x64, ARM64 |

- 컴파일러: Visual Studio 2026(MSVC) 기본. CI에서 VS 2022(`windows-2022`)로도 빌드해 호환을 확인한다.
- 7-Zip의 clang-cl 지원(`CLANG_FLAGS_TARGET`)은 유지하되 공식 빌드는 MSVC로 한다.

---

## 6. 배포와 서명

| 항목 | 결정 |
|---|---|
| 패키지 | MSIX / MSIX 번들 (x64, ARM64) |
| 채널 | ① GitHub Releases(서명된 사이드로드 MSIX) ② Microsoft Store ③ WinGet(①의 MSIX를 가리킴) |
| 서명 | 사이드로드: **Azure Trusted Signing** 등 공개 CA 인증서. Store: Microsoft가 재서명. 개발용: 자체 서명 인증서를 "신뢰할 수 있는 사람"에 등록 |
| 등록 항목 (매니페스트) | `windows.fileExplorerContextMenus`(셸 확장, `comServer`), `uap:FileTypeAssociation`(지원 확장자), 앱 알림, 실행 별칭(`jzip` — 7-Zip이 함께 설치된 PC와 충돌하지 않도록 `7z`는 등록하지 않음) |
| 기본 앱 | 사용자가 Windows 설정에서 선택. 첫 실행·S8-07에서 `ms-settings:defaultapps`로 안내 |
| 업데이트 | Store: 자동 / 사이드로드: `.appinstaller` 자동 업데이트 / WinGet: `winget upgrade` |
| 제외 | 포터블 zip, MSI, sparse package |

---

## 7. 의존성

### Microsoft 구성 요소 (NuGet)

| 패키지 | 버전 기준 | 용도 | 라이선스 |
|---|---|---|---|
| Microsoft.WindowsAppSDK | 2.5.x (2.5.1, 2026-09-16) | WinUI 3, 앱 알림, MSIX 도구 | MIT |
| Microsoft.Windows.CppWinRT | 3.0.x (3.0.260818.1) | C++/WinRT 투영 | MIT |
| Microsoft.Windows.ImplementationLibrary (WIL) | 최신 안정판 | RAII 핸들, HRESULT 처리 | MIT |

### Windows API (추가 설치 없음)

| 기능 | API |
|---|---|
| #17 인코딩 감지 | `MultiByteToWideChar(MB_ERR_INVALID_CHARS)` + 자체 점수 알고리즘 (ICU·uchardet 미사용) |
| #21 악성코드 검사 | AMSI, `IAttachmentExecute` |
| #23 암호 관리자 | DPAPI(`CryptProtectData`), Windows Hello(`Windows.Security.Credentials.UI.UserConsentVerifier`) |
| #26 썸네일 | WIC (Windows Imaging Component) |
| #31 휴지통 | `IFileOperation` (`FOFX_RECYCLEONDELETE`) |
| #32 완료 알림 | `Microsoft.Windows.AppNotifications.AppNotificationManager` |
| #28 다크모드, #29 DPI | WinUI 3 테마(`RequestedTheme`)와 모니터별 DPI 기본 지원 — 비공개 uxtheme API 불필요 |

### 서드파티 (소스 동봉 `third_party/`)

| 라이브러리 | 버전 | 용도 | 라이선스 | 비고 |
|---|---|---|---|---|
| LZ4 | 1.10.0 | #8 LZ4 해제 | BSD-2-Clause | 7-Zip ZS(LGPL)가 같은 방식으로 포함 — 핸들러 구조 참고 |
| unalz | 최신 | #7 구형 ALZ 해제 | zlib | 구 ALZ만 지원 |
| GoogleTest | vcpkg 기준 최신 | 단위 테스트 (배포물 미포함) | BSD-3-Clause | vcpkg 매니페스트 |

### 직접 구현

| 대상 | 참고 | 비고 |
|---|---|---|
| EGG 해제 (#7) | `alkegi/unegg`(Rust, MIT) 구현을 읽고 C++로 새로 작성 | 공식 스펙 없음. AZO 압축, LEA 암호도 직접 구현 필요. 반디집 공식 목록에 EGG는 없음(ALZ만) → 우선순위 재검토 대상 |
| 손상 ZIP 복구 (#24) | 7-Zip `ZipIn.cpp` `ReadLocals` | 복구 사본 저장만 추가 |

라이선스 원칙: j-zip 전체는 7-Zip을 따라 **GNU LGPL**(unRAR 제한 승계). 동봉 라이브러리는 LGPL과 호환되는 허용형(MIT·BSD·zlib)만 쓰고, `THIRD_PARTY_NOTICES.md`에 고지한다.

---

## 8. 테스트와 CI

| 단계 | 도구 | 대상 |
|---|---|---|
| 단위 테스트 | GoogleTest | `jzip-core`(인코딩 감지, 스마트 해제 계획, 경로 계산), 새 포맷 파서(ALZ/EGG/LZ4) |
| 통합 테스트 | 샘플 아카이브 코퍼스 + `7z.exe` 실행 스크립트 | 각 기능 명세 8절 수용 기준의 입력(한글 ZIP, 분할, 암호, 손상, tar.gz 등) |
| 패키지 테스트 | CI에서 MSIX 설치 → 셸 등록·파일 연결 확인 | #19, 기본 앱 후보 등록 |
| UI 테스트 | 수동 체크리스트(05 명세 8절) | UI 자동화 도구는 시제품 단계에서 결정 |

CI 매트릭스 (GitHub Actions):

| 러너 | 컴파일러 | 아키텍처 | 역할 |
|---|---|---|---|
| `windows-2025` | VS 2026 | x64 | 기본 빌드·테스트·MSIX |
| `windows-2022` | VS 2022 | x64 | 컴파일러 호환 확인 |
| `windows-11-arm` | VS 2026 | ARM64 | 네이티브 ARM64 빌드·테스트 |

---

## 9. 위험과 선행 검증(스파이크)

본격 개발 전에 아래를 작은 시제품으로 먼저 확인한다.

| ID | 확인할 것 | 실패 시 대안 |
|---|---|---|
| SP-1 | WinUI 3 MSIX에 `jzip-shell.dll`을 `windows.fileExplorerContextMenus`로 등록 → Win11 새 메뉴에 표시되는지. Win10 2004와 Win11의 "추가 옵션 표시"(예전 메뉴)에서 어떻게 보이는지 | 예전 메뉴용 `IContextMenu` 등록 방식 별도 검토 |
| SP-2 | 아카이브 항목 10만 개를 WinUI 3 목록(`ItemsView`/`ListView`)으로 가상화했을 때 스크롤·정렬 성능 | 목록만 자체 가상화 컨트롤로 교체 |
| SP-3 | 셸 확장 → `JZip.exe` 실행 인자 전달(현재 7zG 방식)과 다중 인스턴스 처리, 같은 창에 탭으로 열기(#36) | 단일 인스턴스 + 명명된 파이프 |
| SP-4 | 관리자 권한으로 실행 시 앱 알림이 조용히 실패하는 문제(조사 결과) | 관리자 실행 시 진행창 유지(S4-12) |
| SP-5 | `uap:FileTypeAssociation`으로 40+ 확장자 등록 시 설치·기본 앱 설정 동작 | 주요 확장자만 등록 |
| SP-6 | Azure Trusted Signing 발급 조건(개인/조직)과 비용 | Store 채널만 우선 |

기타 위험:

- **WinUI 3 + 네이티브 C++ 사례가 C#보다 적다** → 샘플·문서가 C# 위주. 팀 숙련도에 맞춰 SP-1~3에서 생산성 확인.
- **Windows App Runtime 의존** → MSIX 프레임워크 의존성으로 해결되지만 오프라인 설치 환경에서는 런타임 패키지를 함께 제공해야 한다.
- **7-Zip 업스트림 추적 비용** → 7zFM/7zG를 대체하므로 UI 쪽 업스트림 변경은 따라가지 않아도 되지만, `UI/Common`·`UI/Agent` 변경은 계속 병합해야 한다.

---

## 10. 검토했지만 채택하지 않은 선택지

| 선택지 | 제외 이유 |
|---|---|
| Mile.Xaml (NanaZip 방식) | 실제 배포로 검증된 방식이지만 레거시 시스템 XAML 계열이고, 사용자가 WinUI 3를 선택함. 두 스택은 혼용 불가 |
| Win32 유지 + 자체 그리기 | 다크모드가 비공개 API에 의존하고, 탭·미리보기 등 모던 UI 구현 비용이 큼 |
| Qt 6 | 배포 크기와 LGPL 재링크 부담, 네이티브 셸 통합은 어차피 Win32로 따로 필요 |
| WPF / .NET | .NET 런타임 의존과 7-Zip C++ 코드와의 브리지(C++/CLI·P/Invoke) 필요 |
| WebView2 | 웹 런타임 의존, 파일 관리자 UI에는 과함 |
| 포터블 배포 | 우클릭 메뉴(Win11)·기본 앱 등록 불가 — 사용자 기준 미충족 |
| sparse package + MSI | MSIX와 같은 서명 비용에 설치 구성만 복잡 |
