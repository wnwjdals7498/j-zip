# F27. ARM64·포터블·스토어 배포

| 항목 | 내용 |
|---|---|
| 기능 번호 | #27 |
| 우선순위 | P3 |
| 7-Zip 26.04 | △ — ARM64 빌드는 가능, MSI는 x86/x64만, 스토어(MSIX) 패키징 없음 |
| 반디집 참고 | 공식 기능 목록 — Windows 7–11(32비트, x64, ARM64) 지원, Microsoft Store 버전과 포터블 버전 제공 |
| 화면 | 배포 (화면 없음) |
| GUI 요소 | S8-04, S8-05, S8-18 (기능 자체의 화면은 없고, 설치 형태에 따라 이 요소가 바뀜) |
| 디자인 파일 | 해당 없음 (`design/project/FeatureMap.dc.html`의 #27 행에 화면 "배포"로만 표시) |
| 관련 기능 | #19, #22, #29 |

## 1. 목적

ARM64 Windows 사용자, 설치 권한이 없는 사용자, Microsoft Store로 앱을 받는 사용자 모두가 j-zip을 쓸 수 있게 배포 형태를 넓힌다. 특히 MSIX 패키지는 Win11 새 컨텍스트 메뉴(#19)에 필요한 패키지 ID를 함께 제공하므로, 스토어 배포와 #19를 한 작업으로 묶는다.

## 2. 현재 7-Zip 동작

- ARM64 빌드: `CPP/Build.mak:31,60,165,193`에 `PLATFORM == "arm64"` 분기가 있고 `CLANG_FLAGS_TARGET = --target=arm64-pc-windows-msvc`를 쓴다. ARM64 어셈블리 `Asm/arm64/7zAsm.S`, `LzmaDecOpt.S`가 있다 (research #27).
- MSI: `DOC/7zip.wxs:20-34`에 x64/Intel64/Intel 정의만 있고 ARM64 MSI 정의는 없다 (research #27).
- 스토어: MSIX/appx 패키징은 레포에 없다. AppxManifest, sparse package, `GetCurrentPackage`/`IsPackaged`도 0건 (research #27, #19).
- 설치기: `C/Util/7zipInstall/7zipInstall.c`가 셸 확장을 등록한다(`:895-917`, `WriteCLSID` 853행 부근) (research #18, #19).
- 포터블: 설정 저장 위치나 포터블 모드에 대한 확인 결과는 research 자료에 없다. 설정은 `UI/Common/ZipRegistry.cpp`(레지스트리)에 저장된다 (research #13, #23).

## 3. 요구사항

- FR-27-1 x64, x86, ARM64 세 아키텍처로 j-zip 전체(7zFM, 7zG, 7z.exe, 셸 확장 DLL, 포맷 DLL)를 빌드한다.
- FR-27-2 MSI 설치 패키지를 x64, x86, ARM64 세 종류로 만든다. ARM64 MSI는 ARM64 셸 확장 DLL을 등록한다.
- FR-27-3 MSIX 패키지를 만들고 서명한다. MSIX는 #19가 요구하는 패키지 ID와 `desktop4:FileExplorerContextMenus` 선언을 포함한다.
- FR-27-4 MSIX 패키지는 Microsoft Store 제출 요건을 만족하도록 구성한다(제출 자체는 9절 미결).
- FR-27-5 포터블 배포(ZIP 하나)를 제공한다. 포터블 실행 시 레지스트리·셸 확장을 등록하지 않고, 설정은 실행 폴더의 파일에 저장한다.
- FR-27-6 포터블에서는 S7 우클릭 메뉴가 없다는 사실을 S8 탐색기 메뉴 탭(S8-18)에 안내한다.
- FR-27-7 모든 배포물에 LGPL 고지와 변경 내역, 소스 위치를 포함한다(02 문서 5절).
- FR-27-8 언어 파일(`Lang\*.txt`)을 모든 배포물에 포함한다. 소스 레포에는 없으므로 빌드 단계에서 가져온다 (research #22).

## 4. GUI 명세

이 기능 자체의 화면은 없다. 영향을 받는 요소:

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S8-04 | MSIX 설치일 때만 "Win11 새 메뉴 + 추가 옵션" 선택 가능 | "Windows 11 새 메뉴 + 추가 옵션" / "예전 메뉴만" | MSI·포터블에서 새 메뉴를 쓸 수 없으면 비활성 + 이유 표시 |
| S8-05 | 포터블일 때 비활성 | (메뉴 항목 표) | 포터블은 셸 확장이 없으므로 편집 불가 |
| S8-18 | 새 메뉴를 쓸 수 없는 설치 형태, 포터블 | "이 설치 형태에서는 Windows 11 새 메뉴를 쓸 수 없음" / "포터블에서는 우클릭 메뉴를 쓸 수 없음" | 표시만 (C14 정보) |

## 5. 동작 규칙과 예외

- MSI와 MSIX를 같은 PC에 동시에 설치하면 셸 메뉴가 두 번 나올 수 있다. 설치기는 다른 형태가 이미 설치돼 있으면 경고하고 중단한다.
- 포터블 실행 파일을 설치본과 같은 PC에서 실행하면 설치본의 레지스트리 설정을 읽지도 쓰지도 않는다.
- ARM64 PC에서 x64 셸 확장은 탐색기(ARM64 네이티브)에 로드되지 않으므로, ARM64 PC에는 ARM64 패키지를 설치해야 한다.
- 32비트(x86) 패키지는 Windows 32비트용이다. 64비트 Windows에서 x86 패키지만 설치하면 64비트 탐색기에 셸 메뉴가 나오지 않는다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| (포터블 판별 파일) | 없음 | 해당 없음 | 실행 폴더에 포터블 표시 파일이 있으면 포터블 모드. 파일 이름은 9절 미결 |

## 7. 구현 지점

- `CPP/Build.mak` ARM64 분기 (research #27): 빌드 스크립트에서 ARM64 대상 추가.
- `DOC/7zip.wxs`: ARM64 MSI 정의 추가 (research #27).
- `C/Util/7zipInstall/7zipInstall.c`: 설치기 (research #27, #19).
- MSIX/AppxManifest: 새로 만듦. #19 sparse package와 같이 처리 (research #27, #19).
- 포터블 설정 저장: `CPP/7zip/UI/Common/ZipRegistry.cpp`가 설정 저장소이므로 여기서 저장 위치를 분기한다. 정확한 변경 함수는 research 자료에 없음.

## 8. 수용 기준

- [ ] ARM64 Windows 11 PC에 ARM64 MSI를 설치하면 `7zFM.exe`가 작업 관리자에서 ARM64 프로세스로 표시되고, 탐색기 우클릭 메뉴(S7-12 예전 메뉴)에 j-zip 항목이 나온다.
- [ ] x64 PC에 MSIX를 설치하면 Win11 새 메뉴 최상단에 S7-01, S7-02가 나온다(#19 수용 기준과 연계).
- [ ] 포터블 ZIP을 `D:\Tools\j-zip`에 풀고 실행해 설정을 바꾼 뒤, `HKCU\Software` 아래에 j-zip 키가 생기지 않고 실행 폴더에 설정 파일이 생긴다.
- [ ] 포터블로 실행하면 S8 탐색기 메뉴 탭에 "포터블에서는 우클릭 메뉴를 쓸 수 없음" 안내가 보이고 S8-05가 비활성이다.
- [ ] 세 아키텍처 배포물 모두에 `Lang\ko.txt`가 있고 S8-15에서 한국어를 고를 수 있다.
- [ ] MSI가 설치된 PC에서 MSIX를 설치하려 하면 경고가 나온다.

## 9. 범위 밖·미결 사항

- 범위 밖: Windows 7·8 지원 검증(최소 지원 버전은 별도 결정), winget 등 다른 배포 채널.
- 미결: Microsoft Store 제출 계정·서명 인증서 확보.
- 미결: 포터블 모드 판별 방식(표시 파일 이름, 설정 파일 형식). 7-Zip의 기존 포터블 지원 여부는 research 자료에서 확인하지 않았다.
- 미결: MSIX(완전 패키지)로 갈지, MSI + sparse package(외부 위치 패키지)로 갈지. #19와 함께 결정한다.
- 미결: Windows 10에서 MSIX 설치 시 셸 확장이 예전 메뉴에 등록되는지 여부는 확인하지 않았다.
- 06에 반영됨 (S8-18)
