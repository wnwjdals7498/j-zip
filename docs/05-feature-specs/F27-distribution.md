# F27. ARM64·스토어 배포 (MSIX, 포터블 제외)

| 항목 | 내용 |
|---|---|
| 기능 번호 | #27 |
| 우선순위 | P3 |
| 7-Zip 26.04 | △ — ARM64 빌드는 가능, MSI는 x86/x64만, 스토어(MSIX) 패키징 없음 |
| 반디집 참고 | 공식 기능 목록 — Windows 7–11(32비트, x64, ARM64) 지원, Microsoft Store 버전과 포터블 버전 제공. j-zip은 ARM64·스토어 배포만 따르고 포터블은 제외한다(5절) |
| 화면 | 배포 (화면 없음) |
| GUI 요소 | S8-04, S8-18 (기능 자체의 화면은 없고, OS·셸 확장 등록 상태에 따라 이 요소가 바뀜) |
| 디자인 파일 | 해당 없음 (`design/project/FeatureMap.dc.html`의 #27 행에 화면 "배포"로만 표시) |
| 관련 기능 | #19, #22, #29, #32 |

## 1. 목적

x64와 ARM64 Windows 사용자, Microsoft Store·WinGet으로 앱을 받는 사용자 모두가 j-zip을 설치할 수 있게 한다. 배포 형식은 **MSIX 하나**로 정한다. MSIX는 Win11 새 컨텍스트 메뉴(#19)에 필요한 패키지 ID, 기본 앱 등록(`uap:FileTypeAssociation`), 완료 알림(#32), Windows App Runtime 자동 설치를 함께 제공한다. 결정과 근거는 [07. 기술 스택](../07-tech-stack.md) 2절, 6절.

## 2. 현재 7-Zip 동작

- ARM64 빌드: `CPP/Build.mak:31,60,165,193`에 `PLATFORM == "arm64"` 분기가 있고 `CLANG_FLAGS_TARGET = --target=arm64-pc-windows-msvc`를 쓴다. ARM64 어셈블리 `Asm/arm64/7zAsm.S`, `LzmaDecOpt.S`가 있다 (research #27).
- MSI: `DOC/7zip.wxs:20-34`에 x64/Intel64/Intel 정의만 있고 ARM64 MSI 정의는 없다 (research #27).
- 스토어: MSIX/appx 패키징은 레포에 없다. AppxManifest, `GetCurrentPackage`/`IsPackaged`도 0건 (research #27, #19).
- 설치기: `C/Util/7zipInstall/7zipInstall.c`가 셸 확장을 등록한다(`:895-917`, `WriteCLSID` 853행 부근) (research #18, #19).
- 설정은 `UI/Common/ZipRegistry.cpp`(레지스트리)에 저장된다 (research #13, #23).

## 3. 요구사항

- FR-27-1 x64, ARM64 두 아키텍처로 j-zip 전체(`JZip.exe`, `jzip-core.lib`, `jzip-shared.lib`, `jzip-win.lib`, `7z.dll`, `7z.exe`, `jzip-shell.dll`)를 빌드한다. x86(32비트)은 만들지 않는다.
- FR-27-2 아키텍처별 MSIX(`j-zip_{ver}_{arch}.msix`)와 이를 묶은 `.msixbundle`을 만든다. 최소 OS는 Windows 10 22H2(빌드 19045)이며 매니페스트 `TargetDeviceFamily MinVersion=10.0.19045.0`으로 선언한다.
- FR-27-3 MSIX 매니페스트에 `windows.comServer` + `windows.fileExplorerContextMenus`(#19), `uap:FileTypeAssociation`(지원 확장자, S8-07), 앱 알림(#32), 실행 별칭 `jzip`을 선언한다. 7-Zip이 함께 설치된 PC와 충돌하지 않도록 별칭 `7z`는 등록하지 않는다.
- FR-27-4 Windows App Runtime은 MSIX 프레임워크 패키지 의존성(`Microsoft.WindowsAppRuntime 2.x`)으로 설치되게 한다.
- FR-27-5 배포 채널은 ① GitHub Releases(서명된 사이드로드 MSIX, `.appinstaller` 자동 업데이트) ② Microsoft Store ③ WinGet(①의 MSIX를 가리키는 매니페스트) 세 가지다.
- FR-27-6 사이드로드 MSIX는 공개 CA 인증서(Azure Trusted Signing 등)로 서명한다. Store 제출본은 Microsoft가 재서명한다.
- FR-27-7 모든 배포물에 LGPL 고지와 변경 내역, 소스 위치, `THIRD_PARTY_NOTICES.md`를 포함한다(02 문서 5절, 07 문서 7절).
- FR-27-8 번역 리소스를 모든 배포물에 포함한다(#22).
- FR-27-9 포터블 zip, MSI, sparse package는 만들지 않는다(5절).

## 4. GUI 명세

이 기능 자체의 화면은 없다. 영향을 받는 요소:

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S8-04 | Windows 11에서만 활성 | "Windows 11 새 메뉴 + 추가 옵션" / "예전 메뉴만" | Windows 10에서는 비활성 + 이유 표시(#19 FR-19-9) |
| S8-18 | Windows 10, 또는 셸 확장 등록을 확인할 수 없을 때 | 06 S8-18 문구 | 표시만 (C14 정보) |

## 5. 동작 규칙과 예외

- 제외한 배포 형식과 이유(07 문서 10절):
  - 포터블: 설치 경로가 고정되지 않아 Win11 새 메뉴(#19)와 기본 앱 등록을 할 수 없다. 사용자 기준("우클릭 메뉴에 등록되고 기본 앱으로 쓸 수 있어야 함")을 충족하지 못한다.
  - MSI, sparse package + MSI: MSIX와 같은 서명 비용이 들면서 설치 구성(런타임 설치, 패키지 ID 등록)만 복잡해진다.
- ARM64 PC에서 x64 셸 확장은 탐색기(ARM64 네이티브)에 로드되지 않으므로 ARM64 PC에는 ARM64 패키지가 설치되어야 한다. `.msixbundle`은 설치 PC에 맞는 아키텍처를 고른다.
- 오프라인 설치 환경에서는 Windows App Runtime 프레임워크 패키지를 함께 제공해야 한다(07 문서 9절 기타 위험).
- 같은 버전 패키지를 다시 설치하거나 서명 인증서가 신뢰되지 않으면 설치가 실패한다. 개발용 자체 서명 인증서는 "신뢰할 수 있는 사람" 저장소에 등록해야 한다.
- 설정 저장 위치는 MSIX 앱 데이터 경로(사용자별)이며, 패키지 제거 시 함께 지워진다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| 해당 없음 | — | — | 배포 형식은 MSIX 하나이므로 설치 형태 설정이 없다 |

## 7. 구현 지점

- `CPP/Build.mak` ARM64 분기 (research #27): `7z.dll`, `7z.exe` nmake 빌드에 ARM64 대상을 추가하고 MSBuild "Makefile 프로젝트"로 감싸 솔루션에서 함께 빌드한다(07 문서 5절).
- `jzip.sln`: `JZip` 단일 프로젝트 MSIX(`WindowsPackageType=MSIX`), `jzip-shared`, `jzip-core`, `jzip-shell`, 테스트 프로젝트. MSBuild MSIX 패키징으로 `.msix`/`.msixbundle`을 만든다.
- 매니페스트 `Package.appxmanifest`: 새로 만든다. #19와 같이 처리한다.
- CI: GitHub Actions `windows-2025`(VS 2026, x64, MSIX), `windows-2022`(VS 2022 호환), `windows-11-arm`(ARM64 네이티브) (07 문서 8절).
- 7-Zip의 `DOC/7zip.wxs`(MSI)와 `C/Util/7zipInstall/7zipInstall.c`(설치기)는 쓰지 않는다.

## 8. 수용 기준

- [ ] ARM64 Windows 11 PC에 `.msixbundle`을 설치하면 `JZip.exe`가 작업 관리자에서 ARM64 프로세스로 표시되고, 탐색기 우클릭 메뉴에 j-zip 항목이 나온다.
- [ ] x64 Windows 11 PC에 MSIX를 설치하면 Win11 새 메뉴 최상단에 S7-01, S7-02가 나온다(#19 수용 기준과 연계).
- [ ] Windows App Runtime이 없는 깨끗한 Windows 10 22H2 PC에 사이드로드 MSIX를 설치하면 런타임이 함께 설치되고 `JZip.exe`가 실행된다.
- [ ] Windows 10 22H2 미만 PC에서는 설치가 거부된다.
- [ ] 설치 후 Windows 설정 › 기본 앱에서 `.zip`, `.7z`, `.alz`의 기본 앱 후보에 j-zip이 있다.
- [ ] 명령 프롬프트에서 `jzip`으로 `JZip.exe`가 실행되고, 7-Zip이 함께 설치된 PC에서 `7z`는 7-Zip을 가리킨다.
- [ ] `.appinstaller`로 설치한 사이드로드 버전이 새 버전 게시 후 자동으로 갱신된다.
- [ ] `winget install`로 GitHub Releases의 같은 MSIX가 설치된다.
- [ ] 배포물에 한국어·영어 번역이 있고 S8-15에서 한국어를 고를 수 있다.

## 9. 범위 밖·미결 사항

- 범위 밖: Windows 10 22H2 미만·Windows 7·8, x86(32비트), 포터블, MSI, sparse package, 컴퓨터 전체 프로비저닝.
- 미결: Microsoft Store 제출 계정 확보.
- 미결(SP-6, 07 문서 9절): Azure Trusted Signing 발급 조건(개인/조직)과 비용. 실패 시 Store 채널만 우선 연다.
- 미결(SP-5): `uap:FileTypeAssociation`으로 40개 이상 확장자를 등록할 때 설치·기본 앱 설정 동작. 문제가 있으면 주요 확장자만 등록한다.
- 해결: MSIX 전체 패키지로 간다. MSI + sparse package는 쓰지 않는다(07 문서 6절, 10절, #19).
- 해결: 최소 OS는 Windows 10 22H2(빌드 19045), 아키텍처는 x64·ARM64(07 문서 1절). 처음엔 2004(19041)로 정했으나 Windows App SDK 공식 지원 OS 표에 2004가 없어 2026-10-14 사용자 결정으로 올림.
- 미결(SP-1): Windows 10에서 MSIX로 등록한 셸 확장이 예전 메뉴에 표시되는지(#19).
- 06에 반영됨 (S8-18)
