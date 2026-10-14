# F18. 우클릭 메뉴

| 항목 | 내용 |
|---|---|
| 기능 번호 | #18 |
| 우선순위 | — |
| 7-Zip 26.04 | ✅ — `IContextMenu` 기반 예전 스타일 우클릭 메뉴가 있음(Win11에서는 "추가 옵션 표시" 안에만 보임) |
| 반디집 참고 | 공식 기능 목록 — "탐색기 우클릭 메뉴"(Explorer right-click menu) |
| 화면 | S7 탐색기 우클릭 메뉴 |
| GUI 요소 | S7-03, S7-04, S7-05, S7-06, S7-07, S7-08, S7-09, S7-11, S7-12, S7-13, S8-05, S8-06 |
| 디자인 파일 | design/project/ContextMenu.dc.html — 오른쪽 하위 메뉴(다른 위치에 풀기… / 미리보기로 열기 / 압축 파일 테스트 / "사진_2026.7z"로 다시 압축 / 압축 옵션 지정… / 메뉴 항목 편집…)와 왼쪽 메뉴의 하위 메뉴 진입 항목 |
| 관련 기능 | #13, #19, #20, #34, #35 |

## 1. 목적

탐색기에서 아카이브를 열지 않고 바로 풀고, 압축하고, 테스트하는 진입점이다. 7-Zip에 이미 있는 기능이며, j-zip에서는 항목 이름과 구성을 새 GUI(S2·S3·S1)로 연결하고 Win11 새 메뉴(#19)와 같은 항목을 예전 메뉴에도 제공한다.

## 2. 현재 7-Zip 동작

- 구현: `CPP/7zip/UI/Explorer/ContextMenu.cpp` `CZipContextMenu`(`IContextMenu`). 명령 표 `:273-283` `g_Commands`. 메뉴 구성 `QueryContextMenu`(700~1100행대).
- 등록: `Explorer/DllExportsExplorer.cpp:237` `DllRegisterServer`, `RegistryContextMenu.cpp:28-29` `\shellex\ContextMenuHandlers\7-Zip`과 `DragDropHandlers`. 설치기 `C/Util/7zipInstall/7zipInstall.c:895-917`.
- 항목 선택 설정: `FileManager/MenuPage.cpp:49-70` `kMenuItems[]`, 계단식 메뉴 `IDX_SYSTEM_CASCADED_MENU`, 아이콘 `IDX_SYSTEM_ICON_IN_MENU`(#34).
- 사용자 입장: Win10에서는 우클릭 즉시 "7-Zip ▸" 하위 메뉴가 보인다. Win11에서는 "추가 옵션 표시"를 눌러야 보인다.

## 3. 요구사항

- FR-18-1 예전 메뉴(S7-12)에 Win11 새 메뉴와 같은 항목·순서·문구를 제공한다.
- FR-18-2 하위 메뉴(S7-03) 안 항목: S7-04 "다른 위치에 풀기…"(S2 열기), S7-05 "미리보기로 열기"(S1에서 열기), S7-06 "압축 파일 테스트"(S4 → S9), S7-07 "\"{이름}.7z\"로 다시 압축", S7-08 "압축 옵션 지정…"(S3 열기), S7-09 "메뉴 항목 편집…"(S8 탐색기 메뉴 탭).
- FR-18-3 각 항목의 위치(맨 위 / 하위 메뉴 안 / 숨김)는 S8-05 설정을 따른다.
- FR-18-4 아카이브가 아닌 파일·폴더에는 압축 항목만, 아카이브에는 풀기·테스트 항목을 표시한다(기존 7-Zip 판별 규칙 유지).
- FR-18-5 S8-06 "Shift 누를 때만 표시"가 켜져 있으면 Shift를 누른 채 우클릭할 때만 j-zip 항목이 보인다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S7-03 | 항상(설정에 따라) | "j-zip ▸" | 하위 메뉴 |
| S7-04 | 아카이브 | "다른 위치에 풀기…" | S2 |
| S7-05 | 아카이브 1개 | "미리보기로 열기" | S1 |
| S7-06 | 아카이브 | "압축 파일 테스트" | S4 → S9 |
| S7-07 | 아카이브 1개, 7z가 아닌 형식 | "\"{이름}.7z\"로 다시 압축" | 풀어서 7z로 다시 압축 → S4 |
| S7-08 | 파일·폴더 | "압축 옵션 지정…" | S3 |
| S7-09 | 항상 | "메뉴 항목 편집…" | S8 탐색기 메뉴 탭 |
| S7-11 | 일반 파일·폴더 | "{이름}.zip으로 압축", "{이름}.7z로 압축" | 바로 압축 → S4 |
| S7-12 | Win11 "추가 옵션 표시", Windows 10 | 위와 같은 항목 | 예전 메뉴. 등록 방식은 F19 SP-1 결과로 확정 |
| S7-13 | j-zip 하위 메뉴 안 | "해시 값 계산 ▸" | 하위 메뉴로 CRC/SHA 등 해시 계산(7-Zip 기존 기능 유지) |
| S8-05 | 설정 › 탐색기 메뉴 | 항목별 "맨 위에 / j-zip 안에 / 숨김" | 위치 저장 |
| S8-06 | 설정 › 탐색기 메뉴 | "메뉴에 아이콘 표시", "Shift 누를 때만 표시" | — |

## 5. 동작 규칙과 예외

- 여러 개를 골랐을 때는 S7-10(#35), 일반 파일·폴더는 S7-11 규칙을 따른다.
- 폴더와 아카이브가 섞여 선택되면 풀기 항목을 숨긴다(기존 `areDirs` 규칙, `ContextMenu.cpp:800-804`).
- S7-07은 원본 아카이브를 지우지 않는다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| `ContextMenu.*` (F34 6절 키, 기존 `CContextMenuInfo` 값을 처음 실행 때 가져옴) | 기존 기본값 | 탐색기 메뉴 (S8-05, S8-06) | 항목 위치·아이콘·Shift 표시. `settings.json`에 저장([08](../08-architecture.md) 6절) |

## 7. 구현 지점

셸 확장은 `jzip-shell.dll`(7-Zip `UI/Explorer` 기반, XAML 없음)이며, `jzip-shared`·`jzip-win`만 링크한다. 메뉴 명령은 [08. 아키텍처](../08-architecture.md) 5절 규약(`JZip.exe --proto 1 --op <op> …`)으로 `JZip.exe`를 띄운다(현재 7zG가 하던 역할, [07. 기술 스택](../07-tech-stack.md) 3절). 항목별 `--op`: S7-04 `extract --dialog`, S7-05 `open`, S7-06 `test`, S7-07 `recompress --format 7z`, S7-08 `compress --dialog`, S7-09 `settings --page shell-menu`, S7-11 `compress --format zip`/`7z`, S7-13 `hash`. S8 설정 화면은 `JZip.exe`에 새로 만든다.

- `CPP/7zip/UI/Explorer/ContextMenu.cpp` `QueryContextMenu`(700~1100행대), `g_Commands`(`:273-283`): 항목 이름·순서를 바꾼다.
- 메뉴 모델: 항목·표시 조건·3단 위치는 `jzip-shared` 명령 등록부(`src/shared/commands/`)에 둔다. `CPP/7zip/UI/Explorer/ContextMenuFlags.h`, `CPP/7zip/UI/Common/ZipRegistry.h:190` `CContextMenuInfo`(현재는 켜기/끄기 체크 목록)는 기존 값을 가져오는 근거로 쓰고, 값은 `settings.json`의 `ContextMenu.*` 키에 저장한다([08](../08-architecture.md) 6절).
- 등록: 기존 `DllRegisterServer`(`DllExportsExplorer.cpp:237`)와 설치기(`7zipInstall.c:895-917`)의 레지스트리 등록은 쓰지 않고, j-zip CLSID를 MSIX 매니페스트(`windows.comServer`, `windows.fileExplorerContextMenus`)에 선언한다(F19 7절). 예전 메뉴 표시 방식은 SP-1에서 확인한다.

## 8. 수용 기준

- [ ] Win10에서 `a.zip` 우클릭 시 "j-zip ▸" 아래 S7-04~S7-09가 06 문서 순서대로 보인다.
- [ ] Win11에서 "추가 옵션 표시"를 누르면 같은 항목이 보인다.
- [ ] S7-05를 누르면 S1에서 `a.zip`이 열린다.
- [ ] S7-06을 누르면 S4가 뜨고 끝나면 S9 또는 완료 표시가 나온다.
- [ ] `photo.jpg` 우클릭 시 풀기·테스트 항목이 없고 S7-08이 있다.
- [ ] S8-05에서 "압축 파일 테스트"를 숨김으로 바꾸면 다음 우클릭부터 보이지 않는다.
- [ ] S8-06 "Shift 누를 때만 표시"를 켜면 Shift 없이 우클릭할 때 j-zip 항목이 없다.

## 9. 범위 밖·미결 사항

- 범위 밖: Win11 새 메뉴 최상단 표시(#19), 항목 선택 UI 개선의 세부(#34).
- 미결: 7-Zip과 j-zip을 함께 설치했을 때 두 셸 확장이 나란히 보이는지 확인이 필요하다. j-zip은 별도 CLSID를 MSIX 매니페스트에 선언하고 7-Zip의 `shellex` 레지스트리 키를 쓰지 않으므로 등록 키는 겹치지 않는다.
- 해결: 제품 화면의 하위 메뉴 이름은 "j-zip"으로 통일했다(06, 04). ContextMenu·Settings 디자인의 "7-Zip" 표기는 디자인 쪽에서 고친다.
- 06에 반영됨 (S7-13 "해시 값 계산 ▸", S8-05 항목 목록에 포함)
