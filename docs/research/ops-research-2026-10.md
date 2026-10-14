# j-zip 운영 리서치 (2026-10-14 기준)

표기: [확인] = 공식 MS 문서에서 직접 확인, [부분] = 문서가 간접적으로만 뒷받침, [미확인] = 공식 근거 못 찾음(추론/일반 지식).

---

## Q1. 패키지된 IExplorerCommand 핸들러는 어디서 실행되나? package identity가 있나?

- 공식 가이드의 매니페스트 예시는 `com:ComServer` 아래 **`com:SurrogateServer`** + `com:Class Path="ContosoCommand.dll" ThreadingModel="STA"`로 DLL을 등록한다. "File Explorer activates the COM class when it builds the context menu." [확인]
  https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/integrate-packaged-app-with-file-explorer
- `SurrogateServer`는 COM AppID의 `DllSurrogate` 모델에 대응하며, `CustomSurrogateExecutable`을 지정하지 않으면 기본 surrogate(일반 COM 의미상 dllhost.exe)에서 DLL이 호스팅된다. 문서는 "DllSurrogate 값이 없으면 CustomSurrogateExecutable을 쓰지 말라"고만 적는다. [부분]
  https://learn.microsoft.com/en-us/uwp/schemas/appxpackage/uapmanifestschema/element-com-surrogateserver
- 공식 문서 어디에도 "explorer.exe 인프로세스" 또는 "dllhost에서 실행"이라고 명시한 문장은 찾지 못했다. 같은 가이드의 경고("File Explorer loads shell extension code as part of the shell experience; GetTitle/GetState 등을 빠르게 유지")는 인프로세스 가정으로도 읽힌다. 서드파티 PowerShell 모듈(MSIX.ContextMenu.ps1)만 "SurrogateServer = dllhost.exe"라고 말함. [미확인: 실제 프로세스 위치는 Process Explorer 등으로 실측 권장]
- Package identity: 이 확장은 "register the command with app identity"가 전제이며 identity가 있는 패키지(패키지 앱 또는 sparse package)에서만 Win11 메뉴에 노출된다. surrogate 호스팅이면 패키지 컨텍스트에서 활성화되므로 DLL 코드는 package identity를 갖는다고 보는 것이 합리적. [부분]
  https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/integrate-packaged-app-with-file-explorer
  https://blogs.windows.com/windowsdeveloper/2021/07/19/extending-the-context-menu-and-share-dialog-in-windows-11/
- 기타: `desktop4:FileExplorerContextMenus` 최소 OS는 Windows 10 1803(17134). DLL 아키텍처는 로드하는 Explorer 아키텍처와 일치해야 함. 설치/업데이트 후 메뉴가 안 보이면 Explorer 재시작 또는 로그오프. 패키지 COM 서버는 `runFullTrust` 필요. [확인]
  https://learn.microsoft.com/en-us/uwp/schemas/appxpackage/uapmanifestschema/element-desktop4-fileexplorercontextmenus
  https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/app-capability-declarations
- 실무 결론: DLL 안에서 `GetCurrentPackageFullName()` 결과와 `GetModuleFileName(NULL)`을 ETW로 한 번 로깅해 실측하는 것이 가장 확실.

---

## Q2. MSIX 레지스트리/파일 가상화 (full-trust 패키지 데스크톱 앱)

공식 문서: https://learn.microsoft.com/en-us/windows/msix/desktop/flexible-virtualization (updated 2026-01-28) [확인]

- **HKCU 런타임 쓰기**: 앱별/사용자별 private hive로 간다. 읽기는 실제 HKCU와 병합돼 보이지만 "Keys in the virtualized hive are only visible to the app." 앱 제거 시 삭제. 따라서 Explorer/비패키지 프로세스는 볼 수 없다. (같은 package identity로 도는 surrogate가 이 hive를 보는지는 문서 미명시 [미확인])
- **AppData 쓰기(Win10 1809 초과)**: 새로 만든 파일/폴더만 private 경로로 가고 병합되어 보인다(앱만 볼 수 있음). 기존 AppData 파일 수정은 실제 파일에 직접. 읽기는 private 먼저, 없으면 실제 AppData. 제거 시 private 쪽 삭제.
- **Opt-out(전체)**: `unvirtualizedResources` restricted capability + `desktop6:RegistryWriteVirtualization=disabled` / `desktop6:FileSystemWriteVirtualization=disabled` (Properties 아래). 최소 OS Win10 1903(18362). HKCU/AppData 가상화 전체가 꺼지고 제거 시 정리되지 않음.
  https://learn.microsoft.com/en-us/uwp/schemas/appxpackage/uapmanifestschema/element-desktop6-registrywritevirtualization
- **Opt-out(세밀, Win11+)**: `virtualization:RegistryWriteVirtualization/ExcludedKeys`(HKCU 내만), `virtualization:FileSystemWriteVirtualization/ExcludedDirectories`(%USERPROFILE%\AppData 내만). Win11 이상에서만 적용, 구버전은 옛 선언을 따름. 여전히 `unvirtualizedResources` capability 필요. 문서는 전체 해제를 "not a fine-grained tool... often exceeds the requirements"라고 평가.
- **Store 영향**: `unvirtualizedResources`는 restricted capability. 문서: "designed for certain types of desktop PC games that are published by Microsoft and our partners... not intended to be used for other scenarios, because it could compromise the system's ability to uninstall cleanly". desktop6 요소 문서도 동일 취지. Store 제출 시 restricted capability 승인 절차 대상(sideload는 승인 불필요). 일반 앱의 승인 가능성은 낮다고 봐야 함. [확인]
  https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/app-capability-declarations
  https://learn.microsoft.com/en-us/windows/uwp/packaging/app-capability-declarations
- 같은 문서는 "external location 패키지에는 이 capability가 필요"라고도 적음(sparse 예시에도 포함). j-zip은 full MSIX이므로 해당 없음.

---

## Q3. 패키지 앱 <-> 셸 확장 DLL 설정 공유 (최적 방법)

### 후보별 평가

| 방법 | DLL(explorer.exe 또는 surrogate)에서 동작? | 비고 |
|---|---|---|
| **LocalState 내 일반 파일(JSON/INI)을 경로로 읽기** | 가능성 높음 | 앱이 `ApplicationData::Current().LocalFolder()`로 쓴 파일은 `%LOCALAPPDATA%\Packages\<PFN>\LocalState\...`의 실제 파일. DLL이 identity 없이 로드돼도 경로 직접 읽기를 시도할 수 있으나 패키지 폴더 ACL 영향은 문서 미명시. [부분] |
| `ApplicationData.Current.LocalSettings` | identity 컨텍스트에서만 | "Settings data is saved in the Windows registry"(앱 private settings hive). 파일 경로로 읽을 수 없고 API로만 접근 -> DLL이 identity로 안 돌면 불가. [확인: ApplicationData Remarks] |
| `ApplicationData.GetForPackageFamilyName` | - | 해당 API 문서를 찾지 못함(URL 404). ApplicationData 클래스 메서드 표에는 `GetForUserAsync`, `GetPublisherCacheFolder`만 있음. 사용 가능하다고 간주하지 말 것. [확인: 메서드 표에 없음] |
| **Publisher cache folder** (`windows.publisherCacheFolders` + `GetPublisherCacheFolder`) | identity 필요 | 같은 publisher의 여러 패키지 간 공유용. 한 패키지 안의 앱<->DLL에는 과잉. 데이터 관리/버전은 개발자 책임. [부분] |
| 비가상화 HKCU (`unvirtualizedResources`) | 가능(실제 레지스트리) | restricted capability, Store 리스크(Q2). |

출처: https://learn.microsoft.com/en-us/uwp/api/windows.storage.applicationdata , https://blogs.windows.com/windowsdeveloper/2016/05/24/sharing-your-local-app-data (publisher cache, 2016 UWP 시대 글이므로 오래됨 주의)

### 권장 (Q3 답)

1. 설정을 LocalState 안의 **단일 JSON 파일**(예: `settings.json`)로 저장. 앱은 `ApplicationData::Current().LocalFolder()` 기준으로 쓰되 원자적 쓰기(temp 파일 후 rename).
2. DLL은 (a) `GetCurrentPackageFamilyName()`/`GetCurrentPackagePath` 성공 시 identity 기반 경로, (b) 실패(identity 없음) 시 `SHGetKnownFolderPath(FOLDERID_LocalAppData)\Packages\<PFN>\LocalState\settings.json`로 **PFN 하드코딩 폴백** 읽기. 없음/파싱 실패는 안전한 기본값. `GetState/GetTitle`에서는 무거운 IO 금지(캐시 + mtime 확인 정도).
3. 레지스트리 가상화와 Store restricted capability 문제를 모두 피하는 방식이며, DLL이 explorer.exe에 로드되든 surrogate에 로드되든 같은 코드 경로.
4. explorer.exe 인프로세스에서 실제로 읽히는지는 공식 문서로 확정 불가 -> 설치 후 실측 필요 [미확인]. 설정을 핸들러가 읽는 방법을 다룬 MS 공식 샘플/가이드는 찾지 못함.

- 샘플: 공식 가이드와 Windows-classic-samples `ExplorerCommandVerb`는 IExplorerCommand/매니페스트만 다루고 설정 공유 언급 없음. AppModelSamples(microsoft/AppModelSamples, Samples/SparsePackages)는 블로그에서 링크됨(이번에 내용 미열람). NanaZip, PhotoStoreDemo의 설정 읽기 방식은 확인 못함 [미확인]. 서드파티 7zipExplorerExtension은 7-Zip 경로를 레지스트리(HKLM/HKCU)에서 읽는 sparse 패키지 사례(패키지 앱 사례 아님).
  https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/integrate-packaged-app-with-file-explorer
  https://github.com/2xRon/7zipExplorerExtension

---

## Q4. C++/WinRT 3.0 (2026)

- NuGet `Microsoft.Windows.CppWinRT` 3.0.260520.1(최초 3.0, 2026-05-22), GitHub 릴리스 최신 3.0.260818.1(2026-08-19). [확인]
  https://www.nuget.org/packages/microsoft.windows.cppwinrt
  https://github.com/microsoft/cppwinrt/releases
- 릴리스 노트: **pre-standard coroutine(`/await`, `<experimental/coroutine>`) 제거**; "Standard coroutines are available by default starting in C++20"; C++17 프로젝트는 `/await:strict`; 모놀리식 `winrt.ixx` 제거(네임스페이스별 `.ixx`); C++20 module(`import winrt.Windows.Foundation;`) 지원; Lookup/TryLookup/Cancel이 예상 시나리오에서 예외를 안 던짐. [확인]
- **최소 컴파일러/VS 버전은 공식 노트에 명시 없음.** CI는 VS2022(v143)와 VS2026(v145, MSVC Build Tools 14.50) 테스트 -> VS 2022(MSVC 14.4x)는 CI 대상이라 동작 가능성 높으나 최소 버전 공식 보장 문구는 없음. [부분]
- NuGet 빌드 규칙 기본 `LanguageStandard`는 stdcpp17. C++20이 강제는 아니며 표준 코루틴을 쓰려면 C++20(또는 `/await:strict`). [확인]
- 서드파티 CppWinRTPlus 포크의 "C++17 제거"는 MS 공식 아님 -> 근거 제외.
- **WinAppSDK 2.5 C++ 템플릿이 참조하는 C++/WinRT 버전: 공식 문서에서 못 찾음 [미확인]** (새 프로젝트 생성 후 `.vcxproj`의 `Microsoft.Windows.CppWinRT` PackageReference 확인).
- 권장: `/std:c++20` + VS 2022 최신 17.x 또는 VS 2026로 빌드 검증, `<experimental/coroutine>`/`/await` 사용 코드 점검.

---

## Q5. Windows App SDK 지원 수명주기

공식: https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-channels (updated 2026-09-29) [확인]

- 2.5.1 = 현재 Stable (2026-09-16). 표에는 "2.0 / 원 릴리스 2026-04-29 / 최신 패치 2.5.1 / Current / End of servicing 2027-04-29"로 표기.
- Stable 주기: "Major releases no more than every six months (plus minor/patch servicing updates as needed)".
- 서비스 정책: Current(최신 stable) / Maintenance(최신 아님, 중요 수정만) -> Out of Support. 최신 패치 적용 + 지원 OS 사용이 지원 조건. 표 패턴상 서비스 종료는 원 출시 후 약 12개월(1.6: 2024-09-04 -> 2025-09-04, 1.7: 2025-03-18 -> 2026-03-18, 2.0: 2026-04-29 -> 2027-04-29). "12개월"은 표에서 도출한 패턴이며 명문 규정은 아님. 1.8은 Maintenance, 종료일 2026-09-24로 표기(이미 경과).
- SemVer: 2.0부터 SemVer 2.0.0, NuGet 버전 = WinAppSDK 버전, 프레임워크 패키지 패밀리명이 major에 연동(다음 side-by-side는 3.0.0). **breaking change는 major 증가 시에만**. 2.5는 2.0 라인의 minor. [확인]
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-notes/windows-app-sdk-2-0?pivots=stable
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/deployment-architecture
- 최소 OS: "Although the Windows App SDK is currently backward compatible to Windows 10 version 1809, ... supported by Microsoft only on the Windows releases still in support". 2.x 전용 별도 최소 OS 문구는 못 찾음. [확인/부분]
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-channels
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/
- 2.5.1 노트: 매니페스트 확장 `windows.diagnosticServiceModule` 추가(크래시 덤프 수집용). Q7 참조.

---

## Q6. Windows 10 지원 현황 (2026-10-14)

- Windows 10 Home/Pro 22H2(마지막 버전) 지원 종료 2025-10-14. **버전 2004(19041)는 Home/Pro 기준 2021-12-14(표시 12/15 UTC)에 서비스 종료**(20H2 2022-05-10, 21H2 2023-06-13). [확인]
  https://learn.microsoft.com/en-us/lifecycle/products/windows-10-home-and-pro
  (Enterprise/Education 2004 종료일은 별도 페이지, 미열람 [미확인])
- ESU: 소비자 ESU 종료일 **2027-10-12**(MS 지원 페이지). 22H2 기기 대상으로 서술. [확인]
  https://support.microsoft.com/en-us/windows/windows-10-support-has-ended-on-october-14-2025-2ca8b313-1946-43d3-b55c-2b95b107f281
  상용 ESU Year1/2/3 종료: 2026-10-13 / 2027-10-12 / 2028-10-10 (MS Q&A 인용, 2차 경로) [부분] https://learn.microsoft.com/en-us/answers/a/12637085
  Microsoft 365 Apps는 Win10에서 2028-10-10까지 보안 업데이트. [확인]
- 2004 자체는 ESU 대상이 아님(22H2 요건에서 추론). [부분]
- WinAppSDK의 Win10 19041 지원: 지원 OS 표에 Win10 21H2, 22H2, 23H2, 1809가 열거되고 **2004는 없음**. 문서는 "backward compatible to 1809"라고 하므로 19041은 **기술적으로 동작 가능하나 공식 지원 대상은 아님**. [확인/추론]
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/support
- 권장: 매니페스트 `MinVersion` 19041은 가능하나, 지원 정책상 실질 최소는 22H2(19045)로 문서화하는 편이 안전.

---

## Q7. ETW TraceLogging, 롤링 파일 로그, WER LocalDumps

### TraceLogging
공식: https://learn.microsoft.com/en-us/windows/win32/tracelogging/tracelogging-native-quick-start [확인]
- `windows.h` 먼저 include, `advapi32.lib` 링크. VS 2013+ / Win10 SDK.
- `TraceLoggingRegister`는 main/DllMain 등에서 가능, **등록 전 로그는 에러 없이 버려짐**.
- **DLL은 언로드 전 `TraceLoggingUnregister` 필수**(안 하면 크래시 가능, "especially important for DLLs"). DllMain 내 로더락 제약은 일반 주의사항.
- provider 핸들은 모듈 범위. **다른 DLL에 핸들을 넘기지 말 것.** 앱 EXE와 셸 DLL은 각자 provider 정의(같은 GUID 공유 가능).
- `WINVER >= 0x602`면 Win8+ 최적화(Win10 2004 타깃이면 문제 없음).
- 수집: `tracelog -start MySession -f out.etl -guid #<provider-guid>` -> 실행 -> `tracelog -stop MySession` -> `tracefmt -o out.txt out.etl`.
- `wpr -start`로 커스텀 provider 수집하려면 .wprp 프로파일 필요(상세 미확인 [미확인]). `logman create trace -p {GUID}` 대안도 공식 페이지 미열람 [미확인].
- 롤링 파일 로그의 공식 가이드는 없음 [미확인]. 경량 대안(비공식): 앱 쪽에 LocalState\logs 내 N개 x 크기 제한의 단순 순환 파일 로거를 두고, 셸 DLL은 ETW만 사용(핸들러에서 파일 IO 최소화). ETW 세션 순환 모드는 tracelog 옵션으로 가능하나 이번에 문서 확인 못함 [미확인].

### WER LocalDumps
공식: https://learn.microsoft.com/en-us/windows/win32/wer/collecting-user-mode-dumps (updated 2025-03) [확인]
- 키 `HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps`(관리자 필요, 기본 비활성). 값: `DumpFolder`(기본 `%LOCALAPPDATA%\CrashDumps`, REG_EXPAND_SZ), `DumpCount`(기본 10), `DumpType`(0 custom / 1 mini 기본 / 2 full), `CustomDumpFlags`. 하위 키 `\<exe이름>.exe`로 앱별 오버라이드.
- 사용자 지정 DumpFolder는 크래시 프로세스가 쓸 수 있는 ACL 필요. 자체 크래시 리포팅 앱은 미지원. 자동 디버거(AeDebug) 설정 시 덤프 미수집.
- **MSIX 패키지 앱에서의 동작은 공식 문서가 다루지 않음 [미확인].** full-trust(mediumIL) 앱은 일반 Win32 프로세스처럼 처리될 가능성이 높고, 설정은 패키지 외부(관리자)가 쓰는 HKLM 값이라 패키지 가상화 영향 밖이라는 것은 추론. 직접 검증 필요(키 이름은 패키지 내 실행 파일명 기준).
- WinAppSDK 2.5.1 노트: `windows.diagnosticServiceModule` 매니페스트 확장 추가(크래시 덤프 수집 진단 모듈 선언), 노트 문구는 self-contained .NET MSIX 앱 대상. 상세 스키마 페이지는 못 찾음 [부분/미확인].
  https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-notes/windows-app-sdk-2-0?pivots=stable
- 대안: ProcDump, Win11 작업관리자 live dump(위 WER 문서에 언급).

---

## 핵심 요약

1. 핸들러는 매니페스트상 `SurrogateServer`로 등록. identity 컨텍스트에서 활성화될 가능성이 높으나 "dllhost vs in-proc" 명시 문장은 없음 -> 실측 필요.
2. HKCU/AppData 신규 파일 쓰기는 private 가상화. 해제는 `unvirtualizedResources`(게임/파트너용, Store 승인 부담) 필요.
3. 설정 공유는 LocalState JSON 파일 + PFN 경로 폴백이 최선.
4. C++/WinRT 3.0: 표준 코루틴(C++20), CI는 VS2022 v143/VS2026 v145. 최소 버전 명시 없음.
5. WinAppSDK 2.x: SemVer, major 간에만 breaking, 릴리스당 약 12개월 서비스.
6. Win10 2004는 2021년 서비스 종료. WinAppSDK는 1809까지 호환이나 지원은 현재 지원 OS 한정.
7. TraceLogging은 DLL 언로드 전 Unregister 필수. LocalDumps의 MSIX 동작은 공식 미확인.
