# F05. 추가 압축 포맷 생성

| 항목 | 내용 |
|---|---|
| 기능 번호 | #5 |
| 우선순위 | P3 |
| 7-Zip 26.04 | △ — 7z, zip, gz, bzip2, xz, tar, wim 생성 가능. ISO·LZH는 읽기 전용, `.zipx` 확장자 없음, zstd 생성은 빌드에서 꺼짐. tar.gz는 한 번에 만들 수 없음 |
| 반디집 참고 | 공식 기능 목록 — 생성 형식 ZIP, 7Z(LZMA2), ZIPX(XZ), EXE, TAR, TGZ, LZH(LH7), ISO(Joliet), GZ, XZ. 자주 쓰는 형식을 바로 고르게 하는 동작만 참고 |
| 화면 | S3 새로 압축 |
| GUI 요소 | S3-04, S3-18, S3-03, S3-05, S3-06, S3-08, S3-09, S3-10, S3-12, S8-03 |
| 디자인 파일 | design/project/Compress.dc.html — 형식 카드 4개(ZIP "어디서나 열림" / 7Z "가장 작게" / EXE "프로그램 없이 풀림" / TAR.GZ "리눅스·서버용"), 고급 패널 "압축 방식" |
| 관련 기능 | #6, #3, #4, #12 |

## 1. 목적

대부분의 사용자는 ZIP·7Z면 충분하지만, 리눅스 서버용 `tar.gz`처럼 특정 형식이 필요한 경우가 있다. 7-Zip은 `tar.gz`를 만들려면 tar로 한 번, gzip으로 또 한 번 압축해야 한다. j-zip은 자주 쓰는 형식을 카드로 바로 고르게 하고, 7-Zip이 이미 만들 수 있는 나머지 형식도 숨기지 않고 제공한다.

## 2. 현재 7-Zip 동작

- 쓰기 등록(`REGISTER_ARC_IO`)이 있는 핸들러: `Archive/Bz2Handler.cpp:474`, `GzHandler.cpp:1190`, `Tar/TarRegister.cpp:14`, `Wim/WimRegister.cpp:12`, `XzHandler.cpp:1445`, `Zip/ZipRegister.cpp:19`, 7z.
- GUI 형식 표 `g_Formats[]` (`CPP/7zip/UI/GUI/CompressDialog.cpp:271~`): 7z, Zip, GZip, BZip2, xz, Tar, wim. zstd 항목은 주석 처리됨(313-322행 부근).
- 형식별 기능 플래그(`CompressDialog.cpp:236-242`, `:271-350`):

| 형식 | 수준 | 멀티스레드 | 암호 | 이름 암호 | 솔리드 | SFX |
|---|---|---|---|---|---|---|
| 7z | 0–9 | ○ | ○ | ○ | ○ | ○ |
| Zip | 0,1,3,5,7,9 | ○ | ○ | — | — | — |
| GZip | 1,5,7,9 | — | — | — | — | — |
| BZip2 | 1,3,5,7,9 | ○ | — | — | — | — |
| xz | 1–9 | ○ | — | — | ○ | — |
| Tar | 0(저장만) | — | — | — | — | — |
| wim | 0(저장만) | — | — | — | — | — |

- ZIPX: Zip은 `g_ZipMethods`(Deflate, Deflate64, BZip2, LZMA, PPMdZip)로 만들 수 있지만 `.zipx` 확장자는 따로 지원하지 않음 (`CompressDialog.cpp:183-190`, `ZipRegister.cpp:20`).
- ISO, LZH: `Archive/Iso/*`, `Archive/LzhHandler.cpp`에 `IOutArchive`가 없어 읽기 전용.
- zstd 생성: `ZstdHandler.cpp:6`의 `Z7_USE_ZSTD_COMPRESSION`이 주석 처리, `CPP/7zip/Compress/ZstdEncoder.h` 없음.
- tar.gz 한 번에 만들기: research 자료에 근거 없음. `CPP/7zip/UI/GUI/*.cpp`와 `UI/Common/Update.cpp`에서 `tgz`/`tar.gz` 문자열 검색 결과 0건(작성 시 직접 확인).

## 3. 요구사항

- FR-05-1 S3-04는 ZIP / 7Z / EXE / TAR.GZ 카드 4개를 보여 준다. 기본 선택은 ZIP.
- FR-05-2 TAR.GZ 카드를 고르면 한 번의 "압축"으로 `이름.tar.gz` 파일 하나가 만들어진다. 중간 `.tar` 파일이 저장 위치에 남지 않는다.
- FR-05-3 TAR.GZ 결과는 표준 gzip으로 감싼 tar여야 한다. 즉 GNU tar `tar -tzf 이름.tar.gz`로 같은 항목 목록이 나와야 한다.
- FR-05-4 카드 외 형식(TAR, GZ, BZIP2, XZ, WIM, TAR.BZ2, TAR.XZ)은 S3-04 옆 "다른 형식…"(S3-18)으로 고를 수 있다.
- FR-05-5 형식을 바꾸면 S3-03 확장자가 자동으로 바뀐다: `.zip`, `.7z`, `.exe`, `.tar.gz`, `.tar`, `.gz`, `.bz2`, `.xz`, `.wim`, `.tar.bz2`, `.tar.xz`.
- FR-05-6 형식별로 지원하지 않는 옵션(§2 표 기준: 수준, 스레드, 암호, 이름 암호, 솔리드)은 비활성화하고 이유 툴팁을 보여 준다. 예: TAR에서 S3-05는 "저장만"만 활성.
- FR-05-7 GZ·BZIP2·XZ(단일 파일 압축 형식)는 대상이 파일 1개일 때만 고를 수 있다. 2개 이상이거나 폴더면 비활성화하고 "여러 파일은 TAR.GZ 사용"을 툴팁으로 보여 준다.
- FR-05-8 ZIPX, LZH, ISO, ZSTD 생성은 이번 범위에서 제공하지 않는다. 목록에도 보이지 않는다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S3-04 | 항상 | "ZIP 어디서나 열림" / "7Z 가장 작게" / "EXE 프로그램 없이 풀림" / "TAR.GZ 리눅스·서버용" | 선택 시 S3-03 확장자, S3-05~S3-10 활성 여부 갱신 |
| S3-18 | 항상(S3-04 옆) | "다른 형식…": TAR, GZ, BZIP2, XZ, WIM, TAR.BZ2, TAR.XZ | 고르면 형식 카드 선택이 풀리고 드롭다운에 선택 형식 표시. GZ·BZIP2·XZ는 FR-05-7 |
| S3-03 | 항상 | 저장 경로 | 형식 변경 시 확장자만 교체(사용자가 바꾼 이름 부분은 유지) |
| S3-05 | 형식에 따라 일부 비활성 | "저장만 / 빠름 / 보통 / 최고" | 형식이 지원하지 않는 수준은 비활성 + 툴팁 |
| S3-06 | 항상 | "예상 크기 약 …" | 형식 변경 시 갱신 |
| S3-08 / S3-09 | 형식에 따라 비활성 | — | F04 FR-04-5/6 |
| S3-10 | 고급 | "압축 방식" | 형식별 방식 목록(7-Zip `g_*Methods`) |
| S3-12 | 항상 | "압축" | TAR.GZ면 tar → gzip을 한 작업으로 실행, S4에는 작업 1개로 표시 |
| S8-03 | 압축 탭 | "기본 형식" | S3-04 기본 선택 |

## 5. 동작 규칙과 예외

- TAR.GZ 작업 중 취소하면 결과 파일과 임시 파일을 모두 지운다.
- TAR.GZ는 암호를 지원하지 않는다. 암호가 필요하면 7Z 또는 ZIP을 안내한다(F04 FR-04-6).
- TAR.GZ에서 gzip 단계는 단일 스레드다(F03 FR-03-3).
- 분할(#1)은 TAR.GZ에도 적용할 수 있다. 결과는 `이름.tar.gz.001` …
- 기존 아카이브에 추가(갱신)할 때는 형식 선택이 기존 아카이브 형식으로 고정된다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| `Compress.DefaultFormat` | ZIP | 압축 | S3-04 기본 형식 |
| `Compress.DefaultLevel` | 보통 | 압축 | S3-05 기본 수준 |

## 7. 구현 지점

- `JZip.exe`(WinUI 3) S3 화면: 형식 카드(S3-04)와 다른 형식…(S3-18). 형식 목록·기능 플래그는 `jzip-core`로 옮긴다(기존 파일: `CPP/7zip/UI/GUI/CompressDialog.cpp` `g_Formats[]`(271행~)).
- TAR.GZ 한 번에 만들기는 research 자료에 변경 지점이 없다. tar 출력을 gzip 인코더로 바로 넘기는 경로를 새로 설계해야 한다(§9).
- 새 생성 형식(LZH·ISO 등)을 추가하려면 `IOutArchive`를 구현하고 `REGISTER_ARC_IO`로 등록(예: `Archive/LzhHandler.cpp`, `Archive/Iso/IsoRegister.cpp`) 후 `g_Formats[]`에 추가 — 이번 범위 밖.

## 8. 수용 기준

- [ ] 폴더 `자료`(파일 20개, 하위 폴더 3개)를 TAR.GZ로 압축 → `자료.tar.gz` 하나만 생기고 `자료.tar`는 없음
- [ ] 위 결과를 GNU tar `tar -tzf 자료.tar.gz` → 항목 23개가 원본과 같은 경로로 나옴
- [ ] 위 결과를 7-Zip 26.04로 `7z t` → 오류 없음
- [ ] 형식 카드 ZIP → 7Z → TAR.GZ 전환 시 S3-03 확장자가 `.zip` → `.7z` → `.tar.gz`로 바뀜
- [ ] TAR.GZ 선택 시 S3-08 암호, S3-09 비활성 + 툴팁
- [ ] "다른 형식…"에서 GZ 선택 시 대상이 폴더면 비활성 + 툴팁
- [ ] 단일 파일 `log.txt`를 XZ로 압축 → `log.txt.xz` 생성, 7-Zip으로 풀림
- [ ] TAR.GZ 압축 중 취소 → 저장 위치에 결과·임시 파일 없음

## 9. 범위 밖·미결 사항

- 범위 밖: ZIPX 생성. 반디집은 ZIPX(XZ)를 만들지만 7-Zip에는 `.zipx` 생성 경로가 없고, 다른 프로그램과의 호환 범위가 확인되지 않았다.
- 범위 밖: LZH, ISO 생성(`IOutArchive` 새로 구현 필요), ZSTD 생성(인코더 소스 없음).
- 미결: TAR.GZ 한 번에 만들기의 구현 방식(스트림 연결 vs 임시 tar 파일 후 gzip). research 자료에 근거가 없어 설계 검토 필요.
- 미결: GZ·BZIP2·XZ에 여러 파일을 넣으려 할 때 7-Zip 현재 동작(오류 메시지 등)은 research 자료에서 확인되지 않았다. FR-05-7은 j-zip 쪽에서 미리 막는 규칙이다.
- 06에 반영됨 (S3-18). 03 #5 비고에 "TAR.GZ 한 번에 만들기는 신규 작업"을 추가함
