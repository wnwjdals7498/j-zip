# F19. Win11 새 컨텍스트 메뉴

| 항목 | 내용 |
|---|---|
| 기능 번호 | #19 |
| 우선순위 | P2 |
| 7-Zip 26.04 | ❌(사용자 기준) — `IExplorerCommand`는 구현돼 있으나 패키지 ID(sparse package/AppxManifest)와 등록이 없어 Win11 최상위 메뉴에 나타나지 않음 |
| 반디집 참고 | 실사용 UX — Win11 탐색기 새 메뉴 최상위에 압축 프로그램 항목이 바로 보이는 일반적인 패턴. 공식 목록의 "탐색기 우클릭 메뉴"를 Win11 기준으로 확장 |
| 화면 | S7 탐색기 우클릭 메뉴 (설정: S8) |
| GUI 요소 | S7-01, S7-02, S7-03, S7-10, S7-11, S7-12, S8-04, S8-18 |
| 디자인 파일 | design/project/ContextMenu.dc.html — Win11 메뉴 모양의 왼쪽 패널(위쪽 아이콘 줄, "여기에 풀기 (스마트)", "\"사진_2026\\\"에 풀기", 하위 메뉴 진입, 맨 아래 "추가 옵션 표시"). design/project/Settings.dc.html — "메뉴 방식" 라디오(Windows 11 새 메뉴 + 추가 옵션 / 예전 메뉴만) |
| 관련 기능 | #13, #18, #27, #34, #35 |

## 1. 목적

Win11에서는 기존 `IContextMenu` 확장이 "추가 옵션 표시" 안으로 밀려나 클릭이 한 번 더 든다.
j-zip은 패키지 ID를 갖춘 설치로 `IExplorerCommand` 항목을 Win11 새 메뉴 최상위에 등록해, 가장 자주 쓰는 풀기·압축을 우클릭 한 번으로 실행하게 한다. COM 코드는 7-Zip에 이미 있으므로 핵심 작업은 패키징·등록이다.

## 2. 현재 7-Zip 동작

- COM 구현(있음)
  - `CPP/7zip/UI/Explorer/ContextMenu.h:31-39` `CZipContextMenu`가 `IExplorerCommand`, `IEnumExplorerCommand`를 구현한다. `ContextMenu.cpp:1489`에서 `CZipExplorerCommand`는 `CZipContextMenu`의 별칭(`#define`)이다.
  - `ContextMenu.cpp:1469~1840`: `GetTitle`(1631), `GetIcon`(1664), `GetToolTip`, `GetCanonicalName`, `GetState`(1699), `Invoke`(1711), `GetFlags`(1729, `ECF_HASSUBCOMMANDS`), `EnumSubCommands`(1756), `Next`(1783), `Clone`(1832), `LoadItems`(1565, `IShellItemArray`).
  - `CPP/7zip/UI/Explorer/MyExplorerCommand.h`(217행): SDK 대신 쓰는 인터페이스 선언(`IID_IExplorerCommand` 199행). `FileManager/ClassDefs.cpp:12`도 이 헤더를 include한다.
  - 클래스 팩토리: `DllExportsExplorer.cpp:176` `DllGetClassObject`.
- 등록(없음)
  - `DllExportsExplorer.cpp` `DllRegisterServer`와 `C/Util/7zipInstall/7zipInstall.c`는 `CLSID\InprocServer32`와 `shellex\ContextMenuHandlers`만 등록한다.
  - `ExplorerCommandHandler`/`DelegateExecute` 등록, AppxManifest, sparse package(`Identity`, `desktop4:FileExplorerContextMenus`), `GetCurrentPackage`/`IsPackaged` 호출은 레포 전체에서 없다.
- 사용자 입장: Win11 우클릭 첫 메뉴에 7-Zip이 없고 "추가 옵션 표시"를 눌러야 보인다.

## 3. 요구사항

- FR-19-1 j-zip 설치기는 j-zip 셸 확장 DLL을 외부 위치(external location)로 가리키는 sparse package를 등록해, 셸 확장 프로세스에 패키지 ID를 부여한다.
- FR-19-2 sparse package의 AppxManifest는 `desktop4:FileExplorerContextMenus`로 j-zip `IExplorerCommand` CLSID를 파일(`*`)과 폴더(`Directory`) 대상으로 선언한다.
- FR-19-3 패키지는 서명되어야 하며, 설치기는 등록 실패 시 예전 메뉴(S7-12)만으로 계속 동작하고 오류를 설치 로그에 남긴다.
- FR-19-4 Win11 최상위에는 S8-05에서 "맨 위에"로 지정된 항목만 직접 보이고, 나머지는 "j-zip ▸"(S7-03) 하위 명령(`EnumSubCommands`)으로 보인다.
- FR-19-5 기본 최상위 항목: 아카이브 1개 = S7-01 "여기에 풀기 (스마트)", S7-02 "\"{이름}\\\"에 풀기", S7-03. 여러 개 = S7-10. 일반 파일·폴더 = S7-11.
- FR-19-6 각 명령의 `GetState`는 선택 항목 종류에 따라 표시/숨김을 정한다(아카이브가 아니면 풀기 숨김). 판별은 확장자 기준으로 빠르게 하고 파일 내용을 읽지 않는다.
- FR-19-7 `Invoke`는 기존 예전 메뉴와 같은 7zG 명령(풀기·압축·테스트)을 실행한다. 같은 동작에 대해 예전 메뉴와 새 메뉴의 결과가 같아야 한다.
- FR-19-8 S8-04에서 "예전 메뉴만"을 고르면 새 메뉴 항목은 모두 숨김(`ECS_HIDDEN`)이 되고 예전 메뉴는 그대로 유지된다. 패키지 등록 자체는 해제하지 않는다.
- FR-19-9 Win10 이하에서는 sparse package를 등록하지 않고 예전 메뉴만 쓴다. S8-04는 비활성으로 "Windows 11 이상에서만 사용"을 표시한다.
- FR-19-10 제거 시 sparse package 등록, CLSID, 셸 확장 등록을 모두 지운다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S7-01 | Win11 새 메뉴, 아카이브 1개 | "여기에 풀기 (스마트)" | #13 스마트 규칙 → S4 |
| S7-02 | Win11 새 메뉴, 아카이브 1개 | "\"{이름}\\\"에 풀기" | 이름 폴더 → S4 |
| S7-03 | 항상 | "j-zip" + `›` | 하위 명령 열기 |
| S7-10 | 아카이브 여러 개 | "각각 폴더로 풀기 (N개)", "하나로 압축" | #35 |
| S7-11 | 일반 파일·폴더 | "{이름}.zip으로 압축", "{이름}.7z로 압축" | 바로 압축 → S4 |
| S7-12 | "추가 옵션 표시" 안 | 같은 항목 | #18 |
| S8-04 | Win11 이상에서 활성 | "Windows 11 새 메뉴 + 추가 옵션" / "예전 메뉴만" | FR-19-8 |
| S8-18 | 새 메뉴를 쓸 수 없는 설치 형태 | 설치 형태 안내(C14 정보) | 표시만. 문구는 F27 |

## 5. 동작 규칙과 예외

- 탐색기는 `IExplorerCommand` 최상위 항목을 앱당 하나 이상 허용하지만 실제 표시 개수와 순서는 셸이 정한다. 06 문서 순서와 다르게 보일 수 있으며 이를 결함으로 보지 않는다(9절 미결).
- 셸 확장이 느리면 메뉴 전체가 늦게 뜨므로 `GetState`·`GetTitle`은 파일 I/O 없이 50ms 안에 끝나야 한다.
- 이름이 긴 아카이브는 `"{이름}\"에 풀기` 문구에서 기존 `MyFormatNew_ReducedName` 방식으로 줄인다.
- 7-Zip이 함께 설치돼 있어도 j-zip 항목이 독립적으로 보여야 한다(CLSID 분리).
- 설치 권한: sparse package 등록은 사용자 단위로도 가능하다고 알려져 있으나, 설치기 권한 모델은 9절 미결.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| `ContextMenu.Style` | `win11` (Win11), `classic` (Win10 이하) | 탐색기 메뉴 (S8-04) | `win11` = 새 메뉴 + 추가 옵션, `classic` = 예전 메뉴만 |
| 항목 위치 | S7-01·S7-02·S7-11 = 맨 위에 | 탐색기 메뉴 (S8-05) | #34 |

## 7. 구현 지점

- 패키징(신규): sparse package용 AppxManifest와 서명된 MSIX를 새로 만든다. 설치기 `C/Util/7zipInstall/7zipInstall.c`의 `WriteCLSID`(853행 부근)에서 `PackageManager.AddPackageByUriAsync(... ExternalLocation)` 호출을 추가한다. 이 코드는 현재 없다.
- COM: `CPP/7zip/UI/Explorer/ContextMenu.cpp`의 기존 `IExplorerCommand` 구현(1469~1840행)을 그대로 쓰고, `GetFlags`(1729)·`EnumSubCommands`(1756)의 하위 메뉴 구성을 S8-05 설정에 맞춘다. 최상위 항목별로 별도 명령 객체가 필요한지 확인한다.
- 클래스 팩토리: `CPP/7zip/UI/Explorer/DllExportsExplorer.cpp:176` `DllGetClassObject`.
- 배포: #27(스토어·MSIX)과 같은 패키징 작업으로 처리할 수 있다(`DOC/7zip.wxs`, `C/Util/7zipInstall`).

## 8. 수용 기준

- [ ] Win11 23H2 이상에 j-zip을 설치한 뒤 `a.zip`을 우클릭하면 첫 메뉴에 "여기에 풀기 (스마트)", "\"a\\\"에 풀기", "j-zip ›"가 보인다.
- [ ] "j-zip ›" 아래에 S7-04~S7-09가 보인다.
- [ ] `a.zip`, `b.zip`을 함께 우클릭하면 "각각 폴더로 풀기 (2개)"가 첫 메뉴에 보인다.
- [ ] `report.docx` 우클릭 시 "report.zip으로 압축"이 첫 메뉴에 보이고 풀기 항목은 없다.
- [ ] 새 메뉴의 "여기에 풀기 (스마트)" 결과와 "추가 옵션 표시" 안 같은 항목의 결과가 같다(F13 수용 기준 샘플 사용).
- [ ] S8-04를 "예전 메뉴만"으로 바꾸면 다음 우클릭부터 첫 메뉴에 j-zip 항목이 없고 "추가 옵션 표시" 안에는 있다.
- [ ] Win10에서는 S8-04가 비활성이고 예전 메뉴가 바로 보인다.
- [ ] 제거 후 `Get-AppxPackage`에 j-zip sparse package가 없고 우클릭 메뉴에도 j-zip 항목이 없다.
- [ ] 7-Zip 26.04와 함께 설치해도 두 프로그램의 메뉴가 각각 동작한다.

## 9. 범위 밖·미결 사항

- 범위 밖: Microsoft Store 배포 자체(#27), 아이콘 디자인(#29).
- 미결: 코드 서명 인증서와 sparse package 서명 방식(자체 서명 시 설치 PC에 인증서 신뢰 등록 필요 여부).
- 미결: 기존 `IExplorerCommand` 구현이 "최상위 항목 여러 개 + 하위 메뉴 하나" 구성을 그대로 낼 수 있는지. research 자료에는 `ECF_HASSUBCOMMANDS` 단일 루트 구성만 확인됐다. 최상위 항목마다 별도 CLSID가 필요할 수 있다.
- 미결: Win11 셸이 서드파티 최상위 항목의 표시 순서·개수를 어떻게 정하는지는 research 자료 범위 밖이다.
- 미결: NanaZip이 같은 기능을 구현했다고 03 문서에 표시돼 있으나(**N**) 그 구현을 참고·이식할지는 라이선스 확인 뒤 정한다.
- 미결: MSI 설치(관리자)와 사용자 단위 설치에서 sparse package 등록 범위.
- 해결: research 자료 머리말에 `ContextMenu.cpp:1489` `#define CZipExplorerCommand CZipContextMenu`로 두 이름이 같은 클래스임을 적었다.
- 해결: 06·04의 S7-11 `기능`을 #18 #34로 고쳤다.
