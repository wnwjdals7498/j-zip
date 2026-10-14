# F28. 다크모드

| 항목 | 내용 |
|---|---|
| 기능 번호 | #28 |
| 우선순위 | P2 |
| 7-Zip 26.04 | ❌ — `FM.cpp`에 다크모드 시도 코드가 주석 처리된 채로 있음("// doesn't work") |
| 반디집 참고 | 실사용 UX — 시스템 다크 테마를 따라 앱 창도 어둡게 바뀌는 일반적인 데스크톱 앱 동작 |
| 화면 | S1 메인 탐색 창, S8 설정 (적용 범위는 S1–S9 전체) |
| GUI 요소 | S1-22, S8-09, S8-14 |
| 디자인 파일 | design/project/Main.dc.html — `dark` 속성(boolean). 켜면 `bg #1F2228`, `chrome #181B20`, `surface #262A31`, `border #3A3F48`, `text #E7E9EC`, `muted #A4ADB8`, `accent #3B82F6`, `accentSoft #1E2C44`, `accentText #BFD5FF`, `sel #24364F`, `thumb #30353D`로 바뀜. design/project/Settings.dc.html — 왼쪽 분류 "화면·언어"(`#28 #22` 배지) |
| 관련 기능 | #22, #29 |

## 1. 목적

Windows를 다크 테마로 쓰는 사용자에게 j-zip 창만 밝게 떠서 눈이 부시고 일관성이 깨진다. 라이트 / 다크 / 시스템 설정 따르기 중 하나를 고를 수 있게 하고, 메인 창부터 대화상자, 진행창까지 같은 테마로 그린다.

## 2. 현재 7-Zip 동작

- `CPP/7zip/UI/FileManager/FM.cpp:214-240`(typedef)와 `:322-385`(`InitInstance` 안, 창 생성 직후)에 `AllowDarkModeForWindow`(uxtheme ordinal 133), `SetPreferredAppMode(ForceDark)`(ordinal 135), `SetWindowTheme`, `DwmSetWindowAttribute(DWMWA_USE_IMMERSIVE_DARK_MODE=20)` 호출이 **전부 `/* */` 주석 처리**돼 있고, 위에 "// doesn't work"라고 적혀 있다 (research #28).
- 사용자 입장: Windows 테마와 관계없이 7zFM, 7zG 대화상자, 진행창이 모두 밝은 표준 Win32 모양으로 보인다.
- 리스트뷰·대화상자의 색을 바꾸는 `WM_CTLCOLOR*` 처리는 `Panel.cpp`, `CPP/Windows/Control/Dialog.cpp`에 없다 (research #28 변경 지점).

## 3. 요구사항

- FR-28-1 S8-09 "화면·언어" 탭에 테마 선택(S8-14: 라이트 / 다크 / 시스템 설정 따르기)을 두고, 기본값은 "시스템 설정 따르기"이다.
- FR-28-2 "시스템 설정 따르기"에서는 Windows 앱 모드(밝게/어둡게)를 읽어 적용하고, 실행 중에 Windows 설정이 바뀌면 다시 시작하지 않고 반영한다.
- FR-28-3 다크 테마에서 모든 색은 04 문서 디자인 토큰의 "다크" 열 값을 쓴다. 컨트롤 코드에 색을 직접 쓰지 않고 토큰 이름으로 참조한다(06 문서 3절).
- FR-28-4 적용 범위: S1 메인 창(제목 표시줄, 툴바, 경로 줄, 트리, 목록과 열 머리, 상태 줄, 미리보기 창), S2–S6·S8·S9 대화상자, S4 진행창. 테마가 적용되지 않은 밝은 컨트롤이 하나도 남지 않아야 한다.
- FR-28-5 제목 표시줄은 다크 테마에서 어둡게 그린다.
- FR-28-6 다크 테마에서도 텍스트 대비 4.5:1 이상을 지킨다(06 문서 접근성 규칙). `success`/`warning`/`danger`의 다크 값은 이 기준으로 정한다(9절).
- FR-28-7 Windows 고대비 모드가 켜져 있으면 j-zip 테마를 쓰지 않고 시스템 고대비 색을 따른다.
- FR-28-8 S8-14에서 테마를 바꾸고 "저장"하면 열려 있는 S1 창에 즉시 반영된다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S8-14 | 항상(화면·언어 탭, S8-09) | "테마": "라이트" / "다크" / "시스템 설정 따르기" | 선택 후 저장 = 모든 열린 j-zip 창에 즉시 적용 |
| S1-22 | 항상 | (표시 문구 없음) | 현재 테마로 S1 전체를 그림. 디자인의 `dark` 속성이 다크 상태 |

S4-10 완료 알림(C21)은 Windows 토스트이므로 Windows 테마를 따르며, 이 설정의 영향을 받지 않는다.

## 5. 동작 규칙과 예외

**WinUI 3 테마 적용 방식** (UI 기술은 WinUI 3로 확정, [07. 기술 스택](../07-tech-stack.md) 7절)

| 대상 | 방식 | 비고 |
|---|---|---|
| 앱 전체 테마 | 각 창 루트 요소의 `RequestedTheme`(`Light` / `Dark` / `Default`). "시스템 설정 따르기"는 `Default`로 두어 Windows 앱 모드를 따름 | 공개 API. 실행 중 Windows 설정 변경도 자동 반영 |
| 색 | 04 토큰을 XAML 테마 리소스(`ThemeDictionaries`의 `Light`/`Dark`/`HighContrast`)로 정의하고 컨트롤은 `ThemeResource`로 참조 | 06 문서 3절의 "토큰 이름으로 참조" 규칙과 같음 |
| 제목 표시줄 | WinUI 3 창 제목 표시줄(`ExtendsContentIntoTitleBar` 또는 기본 제목 표시줄)이 테마를 따름 | 비공개 API 불필요 |
| 목록(S1-14)·트리(S1-13)·탭·메뉴·체크박스·드롭다운·스크롤바 | WinUI 3 기본 컨트롤이 테마 리소스로 그림 | 직접 그리기 불필요 |
| 고대비 | `HighContrast` 테마 리소스가 시스템 고대비 색을 따름 | FR-28-7 |

채택하지 않은 방식(기록): 7-Zip 26.04의 주석 코드처럼 Win32 창에 uxtheme 비공개 함수(`AllowDarkModeForWindow`, `SetPreferredAppMode` — 이름 대신 서수로만 노출)와 `WM_CTLCOLOR*`·직접 그리기를 조합하는 방식은 쓰지 않는다. 공개 문서는 제목 표시줄(`DWMWA_USE_IMMERSIVE_DARK_MODE`)까지만 다루고, 공용 컨트롤·메뉴는 비공개 API에 의존해 Windows 빌드마다 동작이 바뀔 수 있으며, 체크박스 라벨·그룹 상자·메뉴 등은 직접 그려야 해 범위가 컸다(research `tech-research-2026-10.md` 4절, 07 문서 10절 "Win32 유지 + 자체 그리기").

- 위험 R1: 04 토큰 색과 WinUI 3 기본 컨트롤 스타일(Fluent)이 다를 수 있다. 토큰을 테마 리소스로 덮어쓸 범위(브러시 키 목록)를 시제품에서 정한다.
- 위험 R2: Mica 등 배경 재질을 쓰면 `bg` 토큰 색과 실제 배경이 달라진다. 1차 범위에서는 토큰 단색 배경을 쓴다.
- 위험 R3: 기본 컨트롤로 표현하기 어려운 요소(형식 배지, 상태 칩 C07 등)는 사용자 정의 템플릿이 필요하다. 이때도 색은 `ThemeResource`로만 참조한다.
- 위험 R4: S7 탐색기 우클릭 메뉴는 탐색기가 그리므로 j-zip 테마와 무관하다(Windows 테마를 따름).
- 위험 R5: S4-10 완료 알림은 Windows 앱 알림이므로 Windows 테마를 따른다.
- 실행 중 Windows 테마 변경은 `RequestedTheme=Default`인 요소에 자동 반영된다. 사용자가 S8-14에서 바꾸면 열린 모든 `JZip.exe` 창의 루트 `RequestedTheme`을 바꾼다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| `Theme` | `system` | 화면·언어 (S8-14) | `light` / `dark` / `system` |

## 7. 구현 지점

- `JZip.exe`(WinUI 3): 앱 리소스(`App.xaml` 등)에 04 토큰을 `ThemeDictionaries`(`Light`/`Dark`/`HighContrast`)로 정의하고, 모든 화면(S1–S9, D1–D3)은 `ThemeResource`로만 색을 참조한다.
- `JZip.exe`: 창마다 루트 `RequestedTheme`을 `Theme` 설정값으로 지정하고, S8-14 저장 시 열린 창 전체에 다시 적용한다.
- `JZip.exe` S8 화면·언어 탭(S8-09, S8-14): 새로 만든다. 설정 저장은 `jzip-core`의 설정 저장소.
- 7-Zip 기존 파일은 고치지 않는다. `CPP/7zip/UI/FileManager/FM.cpp`(typedef `:214-240`, 주석 블록 `:322-385`), `Panel.cpp`, `CPP/Windows/Control/Dialog.cpp`는 7zFM의 현재 동작 근거일 뿐이다 (research #28).

## 8. 수용 기준

- [ ] Windows "앱 모드 = 어둡게" + j-zip "시스템 설정 따르기"에서 S1을 열면 창 배경 `#1F2228`, 목록 배경 `#262A31`, 본문 글자 `#E7E9EC`로 그려지고, 제목 표시줄도 어둡다.
- [ ] 실행 중 Windows 앱 모드를 밝게로 바꾸면 5초 안에 S1이 라이트 색으로 바뀐다(재시작 없음).
- [ ] 다크 테마에서 S2(Extract.dc.html 항목 전부), S3, S4, S5, S6, S8, S9, D1–D3을 하나씩 열었을 때 흰 배경 컨트롤이 하나도 없다(체크박스 라벨, 드롭다운 목록, 스크롤바 포함).
- [ ] 다크 테마에서 S1-14 선택 행 배경이 `#24364F`이고 왼쪽 3px 강조선이 `#3B82F6`이다.
- [ ] 대비 검사 도구로 다크 테마의 본문·보조 텍스트·버튼 글자 대비가 모두 4.5:1 이상이다.
- [ ] 고대비 모드를 켜면 j-zip 다크 색이 아니라 시스템 고대비 색으로 그려진다.
- [ ] `JZip.exe`가 uxtheme 비공개 함수를 불러오지 않는다(`dumpbin /imports`와 소스 검색으로 `uxtheme` 서수 호출이 없음을 확인).

## 9. 범위 밖·미결 사항

- 범위 밖: 사용자 정의 색·강조색 선택, S7 탐색기 메뉴의 테마.
- 해결: 04 토큰 표에 성공/경고/오류의 다크 값(`#3FB950` / `#D29922` / `#F85149`)과 `accentText`(다크 `#BFD5FF`), `thumb`(라이트 `#E3E6EA`, 다크 `#30353D`)를 추가했다. FR-28-6 대비 검증은 이 값으로 한다.
- 해결: 메뉴 막대·컨텍스트 메뉴는 WinUI 3 `MenuBar`/`MenuFlyout`이 테마를 따르므로 별도 다크 처리가 필요 없다.
- 해결: 최소 OS는 Windows 10 2004(빌드 19041)로 정했다(07 문서 1절). 제목 표시줄은 WinUI 3 창이 처리하므로 `DWMWA_USE_IMMERSIVE_DARK_MODE` 빌드 범위는 따지지 않는다.
- 해결: UI 기술은 WinUI 3로 확정했다(02 문서 4절, 07 문서). Win32 다크 처리 작업(uxtheme 비공개 함수, `WM_CTLCOLOR*`, 직접 그리기)은 범위에서 빠졌다.
- 미결: 토큰 색과 Fluent 기본 스타일의 차이를 어디까지 맞출지(R1), Mica 사용 여부(R2).
- 06에 반영됨 (5절 "테마·DPI" 규칙: 모든 화면은 토큰만 바꿔 다크를 만든다. 디자인은 Main만 다크를 보여 준다)
