# 02. 개발 계획

## 0. 준비

- **소스**: 7-zip.org 공식 배포 소스 또는 GitHub 미러 `ip7z/7zip`
- **라이선스 준수**
  - GNU LGPL: 수정한 소스 공개, 저작권 표기 유지
  - 일부 BSD-3-Clause 코드 포함
  - unRAR 코드는 RAR 압축기 제작에 사용 금지 (unRAR license restriction)
- **빌드 환경**: MSVC + `nmake`, Windows x64 (이후 ARM64)
- **기존 포크 조사** (중복 개발 방지)
  - NanaZip: Win11 새 컨텍스트 메뉴, 다크모드, MSIX 배포 구현됨
  - 7-Zip ZS: zstd, brotli, lz4 등 코덱 추가

## 1. 소스 구조

| 모듈 | 경로 | 역할 |
|---|---|---|
| 7zFM | `CPP/7zip/UI/FileManager` | 파일 관리자 GUI |
| 7zG | `CPP/7zip/UI/GUI` | 압축/해제 대화상자, 진행창 |
| 셸 확장 | `CPP/7zip/UI/Explorer` | 탐색기 우클릭 메뉴 |
| 공통 로직 | `CPP/7zip/UI/Common` | 해제 경로, 옵션 처리 |
| 포맷 | `CPP/7zip/Archive` | zip, 7z, rar 등 파서 |

## 2. 개선 대상

기능 목록과 우선순위는 [03. 기능 벤치마크](03-feature-benchmark.md), 화면 설계는 [04. GUI 레퍼런스](04-gui-reference.md) 참고.

주요 작업 위치:

| 기능 | 작업 위치 |
|---|---|
| #17 파일명 인코딩 자동 감지 | `Archive/Zip` (UTF-8 플래그 없는 항목 판별) + `UI/Common` |
| #12 TGZ 한 번에 풀기 | `UI/Common/Extract*` |
| #13 스마트 해제 | `UI/Common/Extract*`, `UI/Explorer` |
| #7 ALZ/EGG 해제 | `Archive/` 신규 핸들러 (공개 구현체 라이선스 확인 필요) |
| #23 비밀번호 관리자 | 7zG 비밀번호 대화상자 + DPAPI 저장소 |
| #19 Win11 컨텍스트 메뉴 | `IExplorerCommand` 구현 + sparse package / MSIX |
| #26 썸네일, #36 탭 | 7zFM 대규모 수정 |

## 3. 개발 순서

1. 원본 소스를 그대로 빌드해 기준선 확보
2. 기능 하나당 브랜치 하나, 작은 커밋
3. 회귀 테스트: 샘플 아카이브 세트로 CLI `7z t` / `7z x` 자동 검증
   - 한글 파일명 zip (UTF-8 플래그 유/무), 분할 압축, 암호 아카이브, 손상 아카이브, tar.gz
4. 동작 비교: REA로 수정 전/후 빌드의 프로세스 동작(파일·레지스트리)을 캡처해 비교 (특히 셸 확장)
5. GUI 변경은 [04. GUI 레퍼런스](04-gui-reference.md)의 화면 단위로 진행

## 4. UI 기술 결정 (선행 과제)

현재 7-Zip GUI는 Win32 `.rc` 대화상자 기반. 레퍼런스 수준의 개편에는 기술 선택이 먼저 필요하다.

| 선택지 | 장점 | 단점 |
|---|---|---|
| Win32 유지 + 커스텀 드로잉 | 기존 코드에서 점진적으로 변경 가능 | 다크모드·고DPI 처리가 번거로움 |
| WinUI 3 / XAML Islands | 모던 UI 구현이 쉬움, NanaZip과 같은 방향 | 대화상자 재작성 범위가 큼 |

## 5. 배포

- 제품명은 7-Zip과 혼동되지 않게 별도 명칭 사용 (작업명: j-zip)
- 소스 공개 및 변경 내역 명시 (LGPL)
- 설치 패키지: MSI 또는 MSIX (Win11 새 메뉴에는 MSIX가 유리)
