# F06. 자동 해제 EXE

| 항목 | 내용 |
|---|---|
| 기능 번호 | #6 |
| 우선순위 | P3 |
| 7-Zip 26.04 | △ — 7z 기반 SFX만 만들 수 있음. ZIP SFX 없음 |
| 반디집 참고 | 공식 기능 목록 — 생성 형식에 "EXE(자동 해제)" 포함. 압축 프로그램이 없는 사람도 실행만으로 풀 수 있는 결과물을 형식 선택 한 번으로 만드는 동작만 참고 |
| 화면 | S3 새로 압축 |
| GUI 요소 | S3-04, S3-03, S3-07, S3-08, S3-09, S3-12 |
| 디자인 파일 | design/project/Compress.dc.html — 형식 카드 3번째 "EXE / 프로그램 없이 풀림" |
| 관련 기능 | #5, #4, #1 |

## 1. 목적

압축 프로그램이 없는 사람에게 파일을 보낼 때 실행만 하면 풀리는 EXE가 편하다. 7-Zip은 "SFX 아카이브 만들기" 체크박스로 이 기능을 제공하지만 7z 형식에서만 보이고 이름도 어렵다. j-zip은 형식 카드 "EXE" 하나로 고르게 한다.

## 2. 현재 7-Zip 동작

- GUI: `kFF_SFX`는 7z 형식에만 켜짐 (`CPP/7zip/UI/GUI/CompressDialog.cpp:287`). 체크박스 `IDX_COMPRESS_SFX`.
- SFX 모듈: `CPP/7zip/UI/GUI/UpdateGUI.cpp:31` `kDefaultSfxModule = "7z.sfx"`. 모듈 경로가 비어 있으면 실행 폴더 + `7z.sfx`를 씀 (`UpdateGUI.cpp:561-564`).
- SFX 빌드: `CPP/7zip/Bundles/SFXWin/makefile` — `AR_OBJS`는 `SplitHandler.obj`뿐, `7Z_OBJS`는 7z만. SFXCon, SFXSetup도 같은 구성이라 3종 모두 7z 전용. 소스는 `Bundles/SFXWin/SfxWin.cpp`, `SFXCon/SfxCon.cpp`, `SFXSetup/SfxSetup.cpp`, 설치기 SFX는 `C/Util/SfxSetup`.
- 사용자 입장: 7z 형식을 고르고 "Create SFX archive"를 켜야 `.exe`가 만들어짐.

## 3. 요구사항

- FR-06-1 S3-04에서 "EXE"를 고르면 7z 형식 + SFX 모드로 압축하고, S3-03 확장자를 `.exe`로 바꾼다.
- FR-06-2 EXE 결과는 j-zip 설치 없이 Windows에서 실행하면 풀 위치를 묻고 풀어야 한다(7-Zip GUI SFX 모듈 동작).
- FR-06-3 EXE에서 S3-08 암호와 S3-09 "파일 이름도 숨기기"는 7Z와 같이 활성이다. 암호를 걸면 실행 시 암호를 묻는다.
- FR-06-4 EXE에서 S3-07 나눠서 압축은 비활성("자동 해제 EXE는 나눠서 만들 수 없음" 툴팁)이다.
- FR-06-5 S3-05 압축 수준과 S3-10 고급 옵션은 7Z와 같은 범위를 쓴다.
- FR-06-6 SFX 모듈에 7-Zip 이름·아이콘이 그대로 보이지 않게 j-zip 이름으로 바꾼 모듈을 함께 배포한다.
- FR-06-7 SFX 모듈 파일이 설치 폴더에 없으면 EXE 카드를 비활성화하고 "자동 해제 모듈이 없음 · 다시 설치 필요" 툴팁을 보여 준다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S3-04 | 항상. 모듈 없으면 EXE 비활성 | "EXE" / "프로그램 없이 풀림" | 선택 시 SFX 모드 |
| S3-03 | EXE 선택 시 | `이름.exe` | 확장자 자동 변경 |
| S3-07 | EXE 선택 시 비활성 | "나누지 않음" 고정 | 툴팁으로 이유 |
| S3-08 / S3-09 | EXE 선택 시 활성 | — | F04 규칙 |
| S3-12 | 항상 | "압축" | S4로 진행 |

## 5. 동작 규칙과 예외

- 결과 `.exe`는 메일 서비스나 회사 보안 정책에서 차단될 수 있다. EXE 카드 툴팁에 "메일로 보낼 때 차단될 수 있음"을 적는다.
- 기존 아카이브에 파일을 추가하는 경로에서는 EXE를 고를 수 없다(새로 만들 때만).
- SFX에 코드 서명은 하지 않는다. 실행 시 Windows SmartScreen 경고가 나올 수 있다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| 해당 없음 | — | — | SFX 모듈은 설치 폴더의 고정 파일을 쓴다 |

## 7. 구현 지점

- `CPP/7zip/UI/GUI/CompressDialog.cpp:287` `kFF_SFX`, `IDX_COMPRESS_SFX` — EXE 카드와 연결.
- `CPP/7zip/UI/GUI/UpdateGUI.cpp:31` `kDefaultSfxModule`, `:561-564` 모듈 경로 결정 — j-zip 모듈 이름으로 교체.
- 모듈 소스: `CPP/7zip/Bundles/SFXWin/SfxWin.cpp`(리소스·문구 교체 대상).
- ZIP SFX(범위 밖)를 만들 경우: `Bundles/SFXWin/makefile`에 Zip 핸들러 객체 추가, `g_Formats` Zip 항목에 `kFF_SFX`, `UpdateGUI.cpp` 모듈 선택 변경.

## 8. 수용 기준

- [ ] 폴더 `자료`를 EXE로 압축 → `자료.exe` 생성
- [ ] j-zip·7-Zip이 없는 깨끗한 Windows 11 VM에서 `자료.exe` 실행 → 풀 위치를 묻고 원본과 같은 파일이 풀림
- [ ] 암호 `Test1234` + EXE → 실행 시 암호를 묻고, 틀리면 풀리지 않음
- [ ] EXE 선택 시 S3-07 비활성, S3-09 활성
- [ ] 생성된 `.exe`의 창 제목·아이콘에 "7-Zip" 문구가 없음
- [ ] 설치 폴더에서 SFX 모듈을 지우고 S3 열기 → EXE 카드 비활성 + 툴팁

## 9. 범위 밖·미결 사항

- 범위 밖: ZIP 기반 SFX, 설치기형 SFX(SFXSetup) 선택 UI, 코드 서명.
- 미결: `7z.sfx`가 `Bundles/SFXWin` 빌드 결과물이라는 대응 관계는 research 자료에 직접 적혀 있지 않다(모듈 이름과 SFXWin 소스가 따로 나옴). 빌드 확인 필요.
- 미결: 7-Zip에서 SFX와 분할을 함께 켰을 때의 동작은 research 자료에서 확인되지 않았다. FR-06-4는 혼란을 막기 위한 j-zip 쪽 제한이다.
- 미결: SFX 모듈 리브랜딩 범위(창 문구, 아이콘, 버전 정보)와 LGPL 고지 위치.
