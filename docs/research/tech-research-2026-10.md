# j-zip 기술 스택 조사 (기준일: 2026-10-14)

표기: **[확인]** = 출처 페이지에서 직접 확인, **[추정/해석]** = 간접 근거 또는 내 판단, **[미확인]** = 확인 못함. 라이선스 해석은 법률 자문이 아님.

---

## 1. NanaZip

| 항목 | 내용 | 출처 |
|---|---|---|
| 최신 릴리스 | **7.0.1843.0** ("NanaZip 7.0, version 2609.2"), 2026-09-17 게시 [확인]. main 브랜치 매니페스트는 7.0.1845.0 (미출시 개발 버전) [확인] | https://api.github.com/repos/M2Team/NanaZip/releases/latest , https://raw.githubusercontent.com/M2Team/NanaZip/main/NanaZipPackage/Package.appxmanifest |
| 7-Zip 기반 | README가 "7-Zip 26.04" 기능 계승이라고 기술 [확인]. (7-Zip 26.04 자체는 2026-10-06 릴리스) | https://github.com/M2Team/NanaZip , https://api.github.com/repos/ip7z/7zip/releases/latest |
| UI 기술 | Modern 에디션 = **XAML Islands 기반 GUI** (repo topics: xaml-islands, xaml-winrt). 자체 툴체인 **Mile.Xaml** 사용. NanaZip Classic은 순수 Win32 GUI. WinUI 3 / Windows App SDK 언급은 **없음** [확인]. Mile.Xaml README: "Windows Runtime XAML 컨트롤용 Windows 11 스타일 + XAML Islands 툴체인", Microsoft.Toolkit.Win32.UI.XamlApplication 기능 통합, .NET 9 SDK의 공식 UWP 지원 기반 → **레거시 시스템 XAML Islands(Windows.UI.Xaml, UWP XAML) 계열**로 판단 [해석] | https://github.com/M2Team/NanaZip , https://github.com/ProjectMile/Mile.Xaml |
| Win11 컨텍스트 메뉴 | **스파스 패키지가 아니라 풀 MSIX 패키지**. 매니페스트에 `windows.comServer`(SurrogateServer, `NanaZip.ShellExtension.dll`, STA) + `desktop4:FileExplorerContextMenus`(`windows.fileExplorerContextMenus`)로 `*`/`Directory`/`Drive` 등록. `runFullTrust`, `Windows.FullTrustApplication`, `unvirtualizedResources` 사용. verb 이름을 `0000NanaZipShellExtension`으로 지어 클래식 메뉴 쪽 표시 문제를 우회(주석 명시). uap10(sparse) 설정은 PackageIntegrity/FileType에만 사용 [확인]. 매니페스트 `TargetDeviceFamily MinVersion=10.0.14393.0`이나 실제 요구 사양은 아래 행 참조 | https://raw.githubusercontent.com/M2Team/NanaZip/main/NanaZipPackage/Package.appxmanifest |
| 다크 모드 | README: "모든 GUI 구성요소에서 다크 모드 지원", 메인 창에 Mica 효과 (다크+HDR 꺼짐일 때 완전 immersive Mica). 구현 세부(소스)는 직접 확인 못함 [미확인] | https://github.com/M2Team/NanaZip |
| 배포 | MS Store 권장, GitHub Releases의 MSIX(msixbundle) 및 portable 패키지(`*_Binaries.zip`), SourceForge, WinGet [확인]. 릴리스 자산: `NanaZip_7.0.1843.0.msixbundle`, `NanaZipPreview_*.msixbundle`, `*_Binaries.zip`, `*_DebugSymbols.zip` | https://github.com/M2Team/NanaZip , https://api.github.com/repos/M2Team/NanaZip/releases/latest |
| 최소 Windows | Windows 10 2004 (빌드 19041) 이상, Windows Server 2022 (20348) 이상; x64, ARM64 (NanaZip 6.0부터) [확인]. | https://github.com/M2Team/NanaZip |
| 라이선스 (자체 코드) | **MIT** (서드파티·7-Zip·7-Zip 파생물 제외). 로고/파일 연결 아이콘은 **CC BY-ND 4.0** [확인] | https://raw.githubusercontent.com/M2Team/NanaZip/main/License.md |
| 7-Zip 부분 | LGPL (v2.1 or later, 7z.dll 이외 대부분), 7z.dll 내 unRAR 제한 조항 LGPL, LZFSE 부분 BSD-3 [확인] | 위와 동일 |
| Mile.* | Mile.Xaml = **MIT** (Kenji Mouri) [확인]. 나머지(Mile.Detours, Mile.Json, Mile.Mobility, Mile.Project.Configurations, Mile.Windows.Helpers, Mile.Windows.Internal, Mile.Windows.UniCrt)는 License.md에 목록만 있고 라이선스 미명시 → 각 repo 확인 필요 [미확인] | https://raw.githubusercontent.com/ProjectMile/Mile.Xaml/main/License.md |

**LGPL 7-Zip 포크로의 코드 재사용 호환성 (판단):**
- MIT 코드는 LGPL-2.1+ 프로젝트에 합류 가능 (MIT는 허용형, 결합물은 LGPL 조건으로 배포). MIT 고지문은 해당 파일/NOTICE에 유지. Mile.Xaml(MIT)도 동일.
- 주의: (1) **로고·아이콘(CC BY-ND)** 은 변경 불가이므로 복사·수정해 쓰면 안 됨 (j-zip 자체 로고 필요). (2) NanaZip 트리 안의 7-Zip 파생/서드파티 파일은 각자 원 라이선스. (3) unRAR 제한 조항(RAR 압축 알고리즘 재구현 금지)은 j-zip 포크도 동일 승계. (4) Mile.* 일부 라이선스 미확인 → 가져오기 전 개별 확인.
- 결론: 라이선스상 대체로 가능하나 위 예외 점검 필요 (법률 자문 아님).

---

## 2. Windows App SDK / WinUI 3

- **최신 안정 버전: Windows App SDK 2.5.1 (2026-09-16)**. 이전 안정: 2.4.0 (08-13), 2.3.1 (07-16), 2.2.0 (06-09), 2.1.3 (05-21), 2.0.1 (04-29). 실험 채널 2.5.4-experimental (09-29), 프리뷰는 2.0 Preview2 (03-31). GitHub 릴리스에는 v1.8.12 (09-24)도 있어 1.8 라인도 서비스 중 [확인]. 출처: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads , https://api.github.com/repos/microsoft/WindowsAppSDK/releases
- 2.0부터 Semantic Versioning, 패키지 패밀리명이 메이저 버전을 따름 (breaking change는 메이저에서만) [확인]. 출처: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-notes/windows-app-sdk-2-0?pivots=stable
- **최소 OS: Windows 10 1809 이상** ("Windows 11 and earlier versions starting from Windows 10, version 1809") [확인]. 출처: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/
- **언패키지드 지원**: 지원됨. 프레임워크 종속(framework-dependent) 방식으로 **Bootstrapper API**(또는 `<WindowsPackageType>None</WindowsPackageType>`)로 런타임 초기화, Dynamic Dependencies 지원. 런타임 배포는 `WindowsAppRuntimeInstall.exe`(x64/x86/arm64, `--quiet`) 또는 MSIX 패키지 직접 설치(기존 MSI/설치 프로그램에 포함 가능) [확인]. **Visual C++ Redistributable 필수**. 출처: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/deploy-unpackaged-apps
  - Self-contained: 2.x 릴리스 노트에 `WindowsAppSDKSelfContained` 속성(클래스 라이브러리 오류 처리 개선)이 언급됨 [확인]. 네이티브 C++ unpackaged self-contained의 상세 지침/지원 범위는 확인하지 못함 [미확인].
  - 런타임 크기: 수치 확인 못함 [미확인]. 구조상 Framework+Main+Singleton+DDLM MSIX가 사용자 PC에 설치되고 x64 PC엔 x86+x64 프레임워크 모두 필요(arm64는 x86/x64/arm64 모두) [확인] → "포터블 단일 폴더" 배포와 맞지 않음 [해석].
- **XAML Islands (Win32 호스팅)**: `Microsoft.UI.Xaml.Hosting.DesktopWindowXamlSource` 가 Windows App SDK의 주 호스팅 API이며 WPF/WinForms/**Win32** 호스트 지원. 패키지 ID 불필요하나 런타임 초기화 선행. 호스트가 직접 처리: STA, DispatcherQueue, `Microsoft.UI.Xaml.Application` 생성, XAML 메타데이터, `WindowsXamlManager.InitializeForCurrentThread`, **`ContentPreTranslateMessage`를 메시지 루프에서 호출(필수)**, 포커스(`NavigateFocus`/`TakeFocusRequested`), per-monitor DPI/테마/접근성 테스트. 공식 샘플은 **언패키지드 .NET 8 WinForms 호스트**뿐이며 네이티브 C++ Win32 워크스루는 확인 못함 [확인/미확인]. 출처: https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/host-controls-existing-desktop-apps (2026-09-02), https://learn.microsoft.com/en-us/windows/windows-app-sdk/api/winrt/microsoft.ui.xaml.hosting.desktopwindowxamlsource?view=windows-app-sdk-2.0
  - 구 "시스템 XAML Islands"(Windows.UI.Xaml, Win10 1903+)는 별개 기술이며 문서가 "혼용 금지"로 명시. 구 문서의 알려진 제약: 런타임 중 다크/라이트 테마 전환에 반응하지 않음, 비패키지 시 VC++ 런타임 필요 등 [확인]. 출처: https://learn.microsoft.com/en-us/windows/uwp/xaml-islands/xaml-islands
- **C++/WinRT 상태**: Wikipedia(2차 출처)는 2022-09 Kenny Kerr가 "maintenance mode"라고 밝혔다고 기술 [2차, 미검증]. 그러나 GitHub 릴리스 API상 **3.0.260520.1 (2026-05-22)** 이 첫 3.0 (C++20 module 지원, `/await` 코루틴 제거 등 breaking), 이후 3.0.260715.1 (07-16), 3.0.260818.1 (08-19) → **현재 활발히 릴리스 중** [확인]. 출처: https://api.github.com/repos/microsoft/cppwinrt/releases , https://en.wikipedia.org/wiki/C%2B%2B/WinRT . Windows App SDK 문서는 WinUI를 "C# 또는 C++ 지원 프레임워크"로 설명 [확인]: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/
- **AppNotificationManager (토스트)**: `Microsoft.Windows.AppNotifications` / `...Builder`. **관리자 권한(elevated) 앱에서는 지원 안 됨 (Show가 조용히 실패)** [확인]. 현재 퀵스타트는 패키지드 앱 기준(Package.appxmanifest의 `windows.toastNotificationActivation` + `windows.comServer`, `Register()`를 `GetActivatedEventArgs()` 이전에 호출)이며 **언패키지드 앱 등록 절차는 해당 페이지에서 확인하지 못함** [미확인]. 단 "패키지 ID 부여(스파스 패키지)" 문서는 토스트/푸시 알림을 ID로 얻는 기능으로 명시 [확인]. 출처: https://learn.microsoft.com/en-us/windows/apps/develop/notifications/app-notifications/app-notifications-quickstart (2026-09-10), https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/grant-identity-to-nonpackaged-apps
  - j-zip 시사점: 7-Zip은 관리자 권한 실행이 흔함 → 토스트가 안 뜰 때의 폴백(트레이 풍선/창 내 알림) 필요 [해석].

---

## 3. 비-스토어 앱의 Win11 컨텍스트 메뉴 (sparse package / external location)

공식 문서 (2026-07-16 갱신): "IExplorerCommand 구현 + 앱 ID로 등록. **패키지드 데스크톱 앱 및 스파스 패키지를 쓰는 언패키지드 Win32 앱**에 적용". https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/integrate-packaged-app-with-file-explorer

요구사항 [확인]:
1. 네이티브 COM DLL이 `IExplorerCommand` 구현 (메뉴 구성 메서드는 빨라야 함, 무거운 작업은 Invoke 이후). DLL 아키텍처는 로드하는 Explorer와 일치.
2. 패키지 매니페스트에 `windows.comServer`(SurrogateServer, STA) + `windows.fileExplorerContextMenus`(`desktop5:ItemType` `*`/`Directory`/`Directory\Background`, `Verb Clsid`).
3. 스파스 패키지 매니페스트: `uap10:AllowExternalContent=true`, `Application`에 `uap10:TrustLevel="mediumIL"` + `uap10:RuntimeBehavior="win32App"`, capability `runFullTrust` + `unvirtualizedResources`.
4. 앱 exe의 사이드바이사이드 매니페스트에 `<msix publisher= packageName= applicationId=>`를 넣어 ID 패키지와 연결 (일치 필수, 불일치 시 `0x80073D54`).
5. 등록: `Add-AppxPackage -Path <msix> -ExternalLocation <설치폴더>` (사용자별) 또는 `Add-AppxPackage -Stage ...; Add-AppxProvisionedPackage -Online` (컴퓨터 전체), 또는 `PackageManager.AddPackageByUriAsync`(`ExternalLocationUri`) / `StagePackageByUriAsync + ProvisionPackageForAllUsersAsync`.
6. 출처: https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/grant-identity-to-nonpackaged-apps (2026-10-03 갱신)

**서명 인증서 필요?** 예. "최종 사용자 컴퓨터에 설치하려면 해당 컴퓨터에서 신뢰되는 인증서로 서명된 ID 패키지 필요"; 매니페스트 `Publisher`는 서명 인증서 Subject와 일치해야 함. 개발용 자체 서명은 공개 인증서를 **Trusted People** 저장소에 넣어야 하고(없으면 `0x800B0109`), 프로덕션은 CA 인증서 또는 **Azure Trusted Signing** 같은 서비스 필요 [확인]. 문서가 서명을 요구하는 대상은 작은 ID용 .msix이며 앱 exe/DLL 서명을 요구한다는 문장은 확인하지 못함 [해석].

**MSI / 포터블 호환성:**
- **MSI/기존 설치 프로그램 재사용 가능**: `AllowExternalContent`의 목적이 기존 설치 프로그램 재사용. 설치 프로그램이 위 등록 명령(PowerShell 또는 PackageManager API)을 호출 [확인].
- **포터블**: 등록 시 `ExternalLocation` 절대 경로가 고정되어야 하고, 경로가 다르면 ID가 사라짐("Identity absent at runtime") [확인]. 따라서 폴더를 옮기는 완전 포터블 모드는 부적합; "설치형에서만 컨텍스트 메뉴 제공" 또는 "첫 실행 시 현재 경로로 등록"이 현실적 [해석].
- `AllowExternalContent`는 빌드 **10.0.19041.0**에서 도입. 매니페스트 `MinVersion=10.0.19041.0` 권장, 이전 OS에선 등록 금지 [확인]. 같은 버전 재등록은 `0x80073CF9` → 먼저 제거 필요.
- 등록/갱신 후 안 보이면 Explorer 재시작 또는 로그아웃/로그인 [확인]. 구 IContextMenu 확장은 "추가 옵션 표시"(클래식 메뉴)에 계속 표시 [확인].
- NanaZip은 스파스가 아니라 풀 MSIX (1장) — j-zip이 비-스토어/MSI 설치를 원하면 스파스 경로가 정답 [해석].

---

## 4. Win32 다크 모드 (공식 API?)

- **공식 문서 범위 (2026-06-16 갱신)**: "Support Dark and Light themes in Win32 apps" — (1) 다크 여부 감지(`UISettings.GetColorValue(UIColorType::Foreground)` + `ColorValuesChanged`), (2) **타이틀바만** `DwmSetWindowAttribute(DWMWA_USE_IMMERSIVE_DARK_MODE=20)`. 문서가 명시: "다크 컬러 세트로 앱 UI를 다시 그리는 방법은 다루지 않는다", Win32 앱은 호환성 때문에 기본 라이트 타이틀바. "전체 UI가 다크 테마를 따라야 완전 지원"이라고만 안내, WinUI 3 등 프레임워크는 자동 [확인]. https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/ui/apply-windows-themes
- **공통 컨트롤/메뉴용 공개 API: 확인하지 못함 (존재 증거 없음)**. `SetPreferredAppMode`(uxtheme ordinal 135), `AllowDarkModeForWindow`(133), `ShouldAppsUseDarkMode`(132), `DarkMode_Explorer` 테마는 여전히 비공개/미문서로 보임. 2020년 ProjectReunion #41에서 문서화 요청된 상태이며 2026 문서화 근거는 못 찾음 [검색 기준; 완전 부정의 증명은 아님]. https://github.com/microsoft/ProjectReunion/issues/41 , https://zenn.dev/tenka/articles/win32_darkmode
- 알려진 한계 (비공식 2차 출처): `DarkMode_Explorer`는 모든 컨트롤을 커버하지 못함(ListView 등), 다단 메뉴 미적용 사례, ordinal은 Windows 빌드별로 달라질 수 있음.
- j-zip 시사점 [해석]: 타이틀바는 공식 API 사용 가능. 컨트롤/메뉴는 owner-draw/custom draw, uxtheme 비공개 ordinal(빌드별 위험), 또는 XAML Islands 중 택일. 공개 API만 고집하면 owner-draw가 유일한 안전 경로.

---

## 5. ALZ / EGG 디코딩

| 항목 | 내용 | 출처 |
|---|---|---|
| **unalz** (원본 C++) | 저자 kippler@gmail.com (kipple.pe.kr), 저작 2004-2009, **zlib License** (상업 이용·재배포·수정 자유, 수정본 표시·원본 오인 금지·고지 유지) [확인]. 구(pre-2009) ALZ만 지원, EGG 미지원 (Ubuntu 패키지 설명) [확인]. 번들 bzip2/zlib 수정 코드의 개별 라이선스는 확인 못함 [미확인] | https://sources.debian.org/src/unalz/0.65-9/main.cpp , https://launchpad.net/ubuntu/noble/+package/unalz |
| ESTsoft 역할 | ALZ/EGG는 ESTsoft ALZip 포맷. **공개 공식 스펙 없음** — `file` 유틸 버그 리포트: "ALZ 공식 문서 없음, unalz 소스를 읽는 수밖에" [확인]. "ESTsoft는 디컴프레션 라이브러리 미제공" 취지의 2003년 분쟁 기술은 위키 미러 2차 자료 [미검증] | https://bugs.astron.com/view.php?id=187 , https://en.wikipedia.org/wiki/ALZip |
| ALZ 사양 | Deflate, Obfuscated Deflate, **bzip2 변형**, ZipCrypto, 분할 | https://en.bandisoft.com/bandizip/help/how-to-open-alz-and-egg-files/ |
| EGG 사양 | Deflate, Bzip2, **LZMA, AZO**; AES128/256, **LEA128/256**; 솔리드, 분할, UTF-8 파일명 | 위와 동일 |
| EGG 스펙 문서 | `dterracino/unegg` repo에 `EGG_Specification.pdf/.zip` 포함 (repo에 라이선스 표기 없음 → 출처/재배포 권리 불명) [확인/미확인] | https://github.com/dterracino/unegg |
| 오픈소스 디코더 | `alkegi/unalz` (Rust, **MIT**, crate v0.2.1 2026-08-02; Store/Deflate/변형 bzip2, ZipCrypto, 분할, CP949→UTF-8; kippler/unalz를 References에 명시, 파생 여부 미명시) / `alkegi/unegg` (Rust, **MIT**; Store/Deflate/Bzip2/LZMA/AZO; ZipCrypto/AES/LEA; 솔리드·분할; CLI 도구로 소개) / `EggDotNet` (C#, NuGet 0.0.6, ALZ/EGG 제한적, "프로덕션 비권장") [확인] | https://github.com/alkegi/unalz , https://github.com/alkegi/unegg , https://docs.rs/crate/unalz/latest , https://feed.nuget.org/packages/EggDotNet/0.0.6 |
| Bandizip "Ark 라이브러리" | Bandizip 도움말에 Ark 라이브러리 언급 **없음**; Bandizip은 프리웨어(라이선스 정책 링크만). 재사용 불가로 간주 [확인/해석] | 위 Bandizip 페이지 |
| libarchive / 7-Zip ZS / NanaZip의 ALZ·EGG | **지원 증거를 찾지 못함** (libarchive 이슈, 7-Zip ZS 코덱 목록(Zstd/Brotli/LZ4/LZ5/Lizard/Fast-LZMA2), NanaZip README 어디에도 없음) [미확인 = "없다"의 증명 아님] | https://github.com/mcmilk/7-Zip-zstd , https://github.com/M2Team/NanaZip |

**LGPL 프로젝트 재배포 가능성 (판단, 법률 자문 아님):**
- unalz(zlib), alkegi/unalz·unegg(MIT)는 허용형 → LGPL-2.1+ 프로젝트에 통합/재배포 가능 (고지 유지). 포맷 자체의 특허/권리는 확인 못함 [미확인].
- 구현 방안 [해석]: (a) 원본 unalz(C++, zlib)를 7-Zip `IInArchive` 핸들러로 래핑 — 구 ALZ만. (b) EGG는 C++ 구현체 공개 확인 못함 → Rust `unegg`를 C++로 포팅하거나 스펙 PDF + alkegi 소스를 참조해 신규 구현 (LZMA는 7-Zip 내장 사용; **AZO와 LEA는 별도 구현 필요**). nmake/MSVC 체인에 Rust 빌드를 넣는 것은 부담.
- `dterracino/unegg`의 스펙 PDF는 라이선스 불명 → repo에 포함하지 말고 링크 참조만 권장.

---

## 6. LZ4

- **라이선스**: LZ4 라이브러리(`lib/`) = **BSD 2-Clause**. `programs/`(CLI) 라이선스는 해당 페이지에서 확인 못함 [미확인]. 출처: https://github.com/lz4/lz4
- **7-Zip ZS (mcmilk/7-Zip-zstd)**: **LZ4 코덱 포함** (codec ID 4F71104, 레벨 1-12, 번들 LZ4 v1.10.0; `lz4-x64.dll` 플러그인 형태도 존재). 최신 **7-Zip ZS 26.04 Release 1 (Zstd 1.5.7)**, 출력 날짜 2026-10-06. 코덱: Zstandard, Brotli, LZ4, LZ5, Lizard, Fast LZMA2. 라이선스: "메인라인 7-Zip과 동일, 대부분 GNU LGPL v2.1-or-later" (COPYING 참조) [확인]. 번들 lz4/zstd/brotli는 각자 라이선스(BSD 계열)일 것으로 보이나 파일별 확인은 안 함 [미확인]. 출처: https://github.com/mcmilk/7-Zip-zstd
- 디코딩 전용이면 LZ4 BSD-2 소스를 포함해도 LGPL 프로젝트와 호환 (BSD 고지 유지) [해석].

---

## 7. GitHub Actions의 MSVC C++ CI (2026)

- **러너 이미지 (runner-images README)**: `windows-latest`/`windows-2025`/`windows-2025-vs2026` (Windows Server 2025, x64), `windows-2022`, **`windows-11-arm`** (arm64), **`windows-11-vs2026-arm`** (arm64 + VS 2026). 폐기 표시 없음. "최대 GA 2개 + 베타 1개 이미지 지원" 정책 [확인]. 출처: https://raw.githubusercontent.com/actions/runner-images/main/README.md
- **VS 2026 전환**: `windows-latest`와 `windows-2025` 라벨은 2026-06-08~06-15 롤아웃으로 **기본 VS 2026**; `windows-2025-vs2026`은 테스트용이며 이전 후 windows-2025를 가리킴; **VS 2022 유지하려면 `runs-on: windows-2022`** [확인]. 출처: https://github.blog/changelog/2026-05-14-github-actions-upcoming-image-migrations . windows-2022 폐기 일정은 해당 글에 없음 [미확인].
- **ARM64**: `windows-11-arm`은 2025-04 공개 저장소 전체(GitHub Free 포함)에 무료 제공 발표 (public preview, **공개 저장소 전용**, 비공개에서는 실패한다는 2차 보고). 2026 시점 GA 여부/비공개 저장소 지원은 확인 못함 [미확인]. 2026-05 기준 GitHub가 arm64 이미지를 직접 관리. 출처: https://blogs.windows.com/windowsdeveloper/2025/04/14/github-actions-now-supports-windows-on-arm-runners-for-all-public-repos/ , 위 changelog. `windows-11-vs2026-arm`은 릴리스 노트상 public preview, 무료 여부 미확인 (https://newreleases.io/project/github/actions/runner-images/release/win25-vs2026%2F20260728.188).
- j-zip 시사점 [해석]: nmake + MSVC는 `vcvarsall.bat` 또는 msvc-dev-cmd 계열 액션으로 환경 구성; VS 2022/2026 x x64/arm64(네이티브 `windows-11-arm`) 매트릭스 가능. 액션 이름/버전은 조사하지 않음 [미확인].
- 부수: C++/WinRT 3.0.260715.1 노트에 "CI를 VS 2022에서 이동" 문구 — 업스트림도 VS 2026 이전 (https://api.github.com/repos/microsoft/cppwinrt/releases).

---

## 8. 대안 비교용 핵심 사실

| 후보 | 핵심 사실 | 출처 |
|---|---|---|
| **Qt 6 (6.12)** | LGPLv3 선택 가능하나 의무: **재링크 가능 메커니즘(사실상 동적 링크)**, 라이선스 사본·Qt 사용 명시, Qt 소스(수정 포함) 제공, 수정 사항 환원 등(페이지 요약). GPL 전용 모듈 존재(Qt Graphs, Quick 3D, Qt GRPC, Qt HTTP Server 등). 상용 라이선스와 혼용 금지. Windows: **Windows 10 1809+ x86_64, 6.12가 Windows 10 지원 마지막 버전**; MSVC 2022 / MinGW 15.1; ARM64 지원(MSVC 2022), ARM64EC는 MSVC 2026이며 설치 패키지 없음. 배포 크기 수치는 미확인. LGPL-2.1+ 포크와 LGPLv3 Qt의 결합 시 v3 의무 검토 필요 [해석] | https://doc.qt.io/qt-6/licensing.html , https://www.qt.io/download-open-source , https://doc.qt.io/qt-6/windows.html |
| **WPF / .NET** | WPF는 Windows 전용, .NET(오픈소스)에서 동작. .NET은 매년 11월 신규 메이저, 짝수=LTS(3년), 홀수=STS(2년, .NET 9부터). 확인한 페이지 갱신일이 2025-11이라 2026-10 현재의 지원 현황은 별도 확인 필요 [확인/미확인]. 프레임워크 종속 시 Desktop Runtime 설치 필요, self-contained는 크기 증가(수치 미확인). 7-Zip C++ 코드와는 C++/CLI 또는 P/Invoke 브리지 필요 [해석] | https://learn.microsoft.com/en-us/dotnet/desktop/wpf/overview/ , https://learn.microsoft.com/en-us/dotnet/core/versions/ |
| **WebView2** | **Evergreen 런타임이 Windows 11에 OS 포함 예정/포함**, 대부분의 Windows 10에도 설치돼 있으나 일부 없음 → 존재 확인 및 부트스트래퍼(~2MB) 필요; 오프라인용 Standalone 설치 프로그램. **Fixed Version은 250MB 이상** 증가; Win10 + 언패키지드 + Fixed v120+는 AppContainer ACL(icacls) 필요; UNC 경로 실행 불가. Win32 C++용 `WebView2Loader.dll`(또는 정적 lib) 동봉 | https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/distribution |
| (참고) **WinUI 3 + XAML Islands** | 2장 참조: 런타임 설치 필요, Fluent/다크/DPI 자동, 호스트가 입력/포커스 책임 | 2장 |

---

## 종합 의견 (조사 기반 판단)

1. NanaZip 방식(Mile.Xaml, 레거시 시스템 XAML Islands 계열)은 검증된 MIT 코드이나 WinUI 3(Windows App SDK)와는 **다른 스택**. Microsoft 문서는 두 스택 혼용을 금지.
2. Win11 컨텍스트 메뉴: 스파스 패키지 + IExplorerCommand가 공식 경로. 서명 필요, MSI 연동 가능, 포터블은 부적합.
3. 다크 모드: 공식은 타이틀바까지. 컨트롤은 owner-draw / 비공개 uxtheme / XAML 중 선택.
4. ALZ/EGG: 허용형 참조 구현(unalz zlib, alkegi MIT) 존재, 스펙 PDF는 출처 불명. libarchive/7-Zip ZS/NanaZip 지원 근거 없음 → 직접 구현 영역 (AZO, LEA 필요).
5. LZ4: BSD-2, 7-Zip ZS에 이미 존재.
6. CI: `windows-latest`=VS 2026, VS 2022는 `windows-2022`, ARM64는 `windows-11-arm`(공개 저장소).