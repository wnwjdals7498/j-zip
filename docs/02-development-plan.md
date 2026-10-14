# 02. 개발 계획

## 0. 준비

- **소스**: 7-zip.org 공식 배포 소스 또는 GitHub 미러 `ip7z/7zip`
- **라이선스 준수**
  - GNU LGPL: 수정한 소스 공개, 저작권 표기 유지
  - 일부 BSD-3-Clause 코드 포함
  - unRAR 코드는 RAR 압축기 제작에 사용 금지 (unRAR license restriction)
- **빌드 환경**: MSVC(Visual Studio 2026 기본, VS 2022 호환 유지). 7-Zip 코어(`7z.dll`, `7z.exe`)는 기존 `nmake`, 새 프로젝트(`JZip.exe`, `jzip-core.lib`, `jzip-shared.lib`, `jzip-win.lib`, `jzip-shell.dll`, 테스트)는 MSBuild 솔루션. nmake 빌드는 makefile 변수로 MSBuild가 만든 `jzip-shared`·`jzip-win`의 `-mt` 변형(`/MT`)을 링크한다. x64 · ARM64. 자세한 내용은 [07. 기술 스택](07-tech-stack.md) 5절
- **기존 포크 조사** (중복 개발 방지)
  - NanaZip: Win11 새 컨텍스트 메뉴, 다크모드, MSIX 배포 구현됨. UI는 Mile.Xaml(시스템 XAML) 기반이라 WinUI 3와 혼용할 수 없으므로 UI 코드는 가져오지 않고, 패키징·셸 등록 방식만 참고한다(로고·아이콘은 CC BY-ND라 사용 금지, [07](07-tech-stack.md) 3절)
  - 7-Zip ZS: zstd, brotli, lz4 등 코덱 추가

## 1. 소스 구조

| 모듈 | 경로 | 역할 |
|---|---|---|
| 7zFM | `CPP/7zip/UI/FileManager` | 파일 관리자 GUI |
| 7zG | `CPP/7zip/UI/GUI` | 압축/해제 대화상자, 진행창 |
| 셸 확장 | `CPP/7zip/UI/Explorer` | 탐색기 우클릭 메뉴 |
| 공통 로직 | `CPP/7zip/UI/Common` | 해제 경로, 옵션 처리 |
| 포맷 | `CPP/7zip/Archive` | zip, 7z, rar 등 파서 |

j-zip 구성 요소와의 대응 ([07. 기술 스택](07-tech-stack.md) 3절):

| j-zip 구성 요소 | 7-Zip 기반 | 역할 |
|---|---|---|
| `JZip.exe` | 7zFM + 7zG를 **대체** | WinUI 3(Windows App SDK 2.5.x) + C++/WinRT 3.0 앱. 화면 S1–S9, D1–D3 전부. 7zFM·7zG의 화면 코드는 동작 근거로만 참고 |
| `jzip-core.lib` | `UI/Common`, `UI/Agent` | 압축·해제·목록 작업 실행 + 설정 서비스(`ZipRegistry` 레지스트리 저장을 `settings.json` 어댑터로 교체) + 현지화 서비스(#22) + 암호 보관(#23) + 앱 전용 Win32 어댑터(DPAPI, AMSI, WIC). WinRT를 참조하지 않는 C++ |
| `jzip-shared.lib` | (신규) | 순수 도메인 로직: 인코딩 후보 점수(#17), 스마트 해제 계획(#13), tar 한 번에 판정(#12), 분할 세트 규칙(#33), 설정 스키마·JSON, 명령 등록부(#18 #34), Lang 파서(#22, `CPP/Common/Lang.cpp` `CLang` 이식). 표준 C++20만, OS 호출 없음. `7z.dll`·`7z.exe`·`jzip-core`·`jzip-shell`이 링크 |
| `jzip-win.lib` | (신규, `src/shared-win/`) | `jzip-shared` 포트의 Win32 구현: 코드페이지 변환기(#17), 파일 시스템 확인(#13), 휴지통 `IFileOperation`(#31), 설정·Lang 파일 읽기(공유 읽기, 마지막 캐시). Win32 API + 표준 C++20, WinRT·XAML 없음. `7z.dll`·`7z.exe`·`jzip-core`·`jzip-shell`이 링크 |
| `7z.dll` | `Bundles/Format7zF` | 포맷·코덱 엔진. ALZ/LZ4 핸들러 추가. `JZIP-PATCH` 훅에서 `jzip-shared`(판단)·`jzip-win`(포트 구현) 호출. nmake 빌드 유지 |
| `jzip-shell.dll` | `UI/Explorer` | `IExplorerCommand` 셸 확장. MSIX `windows.fileExplorerContextMenus`로 등록. `jzip-shared`·`jzip-win`만 링크 |
| `7z.exe` | `UI/Console` | 명령줄. j-zip 로직은 `jzip-shared`·`jzip-win` 링크와 업스트림 `JZIP-PATCH` 훅으로 반영. `settings.json`은 읽지 않고 스위치로만 동작 |

## 2. 개선 대상

기능 목록과 우선순위는 [03. 기능 벤치마크](03-feature-benchmark.md), 화면 설계는 [04. GUI 레퍼런스](04-gui-reference.md) 참고.

주요 작업 위치:

| 기능 | 작업 위치 |
|---|---|
| #17 파일명 인코딩 자동 감지 | `jzip-shared` 점수기 + `jzip-win` 코드페이지 변환기 + `Archive/Zip/ZipItem.cpp` `JZIP-PATCH` 훅(UTF-8 플래그 없는 항목 판별) + `UI/Common` |
| #12 TGZ 한 번에 풀기 | `jzip-shared` 판정 + `UI/Common/OpenArchive.cpp`·`Extract*` 훅 |
| #13 스마트 해제 | `jzip-shared` 계획 + `jzip-win` 파일 시스템 확인 + `UI/Common/Extract*` 훅, `UI/Explorer` |
| #31 해제 후 원본 휴지통 | `jzip-win` 휴지통(`IFileOperation`) + `UI/Common/Extract.cpp` `Extract()` 훅 |
| #22 다국어 | `jzip-shared` Lang 파서(7-Zip `CPP/Common/Lang.cpp` `CLang` 이식) + `jzip-core` 현지화 서비스(`UI/FileManager/LangUtils.cpp` 이식, 파일 로드는 `jzip-win`) + 7-Zip `Lang\*.txt` 재사용. `.resw`는 MSIX 매니페스트 표시 문구만 |
| #7 ALZ 해제 | `Archive/` 신규 핸들러 (unalz(zlib) 래핑). EGG는 지원하지 않음 |
| #23 비밀번호 관리자 | `JZip.exe` S6 화면 + `jzip-core` DPAPI 저장소 |
| #19 Win11 컨텍스트 메뉴 | `jzip-shell.dll`(`IExplorerCommand`) + MSIX 매니페스트 `windows.fileExplorerContextMenus` |
| #26 썸네일, #36 탭 | `JZip.exe` S1 화면(WinUI 3 `TabView`, WIC 썸네일). 7zFM은 수정하지 않고 대체 |

## 3. 개발 순서

1. 원본 소스를 그대로 빌드해 기준선 확보
2. 기능 하나당 브랜치 하나, 작은 커밋
3. 회귀 테스트: 샘플 아카이브 세트로 CLI `7z t` / `7z x` 자동 검증
   - 한글 파일명 zip (UTF-8 플래그 유/무), 분할 압축, 암호 아카이브, 손상 아카이브, tar.gz
4. 동작 비교: REA로 수정 전/후 빌드의 프로세스 동작(파일·레지스트리)을 캡처해 비교 (특히 셸 확장)
5. GUI 변경은 [04. GUI 레퍼런스](04-gui-reference.md)의 화면 단위로 진행

## 4. UI 기술 결정 (확정)

**WinUI 3 (Windows App SDK 2.5.x) + C++/WinRT 3.0, C++20으로 확정했다.** 결정 내용과 근거는 [07. 기술 스택](07-tech-stack.md)에 있다.

- 7zFM과 7zG를 고치지 않고, 독립 WinUI 3 앱 `JZip.exe`가 둘을 대체한다(XAML Islands 아님). 화면은 XAML 페이지·창으로 새로 작성한다
- 7-Zip의 Win32 `.rc` 대화상자 코드는 동작 근거로만 참고하고, 로직은 `jzip-core.lib`로 옮겨 재사용한다
- 다크모드·고DPI는 WinUI 3 기본 기능으로 처리한다(비공개 uxtheme API 사용 안 함)

검토했던 선택지와 제외 이유:

| 선택지 | 결과 | 이유 |
|---|---|---|
| Win32 유지 + 커스텀 드로잉 | 제외 | 다크모드가 비공개 API에 의존하고, 탭·미리보기 등 모던 UI 구현 비용이 큼 |
| XAML Islands / Mile.Xaml (NanaZip 방식) | 제외 | 레거시 시스템 XAML 계열이며 WinUI 3와 혼용 불가 |
| WinUI 3 독립 앱 | **채택** | 모던 UI·다크모드·DPI 기본 지원, MSIX와 함께 우클릭 메뉴·기본 앱·알림 요구를 한 번에 충족 |

본격 개발 전에 G1 단계의 시제품 SP-1~SP-6(07 문서 9절, 단계는 [09](09-development-process.md) 3절)으로 셸 등록, 대용량 목록 성능, 단일 인스턴스 등을 먼저 확인한다.

## 5. 배포

- 제품명은 7-Zip과 혼동되지 않게 별도 명칭 사용 (작업명: j-zip)
- 소스 공개 및 변경 내역 명시 (LGPL)
- 설치 패키지: **MSIX만** 제공한다(x64 · ARM64, Windows 10 22H2(빌드 19045) 이상). 채널은 ① GitHub Releases(서명된 사이드로드 MSIX, `.appinstaller` 자동 업데이트) ② Microsoft Store ③ WinGet(①의 MSIX를 가리킴)
- 포터블 zip, MSI, sparse package는 제공하지 않는다. 포터블은 우클릭 메뉴(Win11)·기본 앱 등록이 불가능하고, sparse package + MSI는 MSIX와 같은 서명 비용에 설치 구성만 복잡하다([07](07-tech-stack.md) 6절, 10절)
