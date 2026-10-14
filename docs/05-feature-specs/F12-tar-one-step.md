# F12. TGZ/TBZ 한 번에 풀기

| 항목 | 내용 |
|---|---|
| 기능 번호 | #12 |
| 우선순위 | P1 |
| 7-Zip 26.04 | ❌ — `.tar.gz`를 풀면 `.tar` 하나만 나오고, 그 `.tar`를 다시 풀어야 함 |
| 반디집 참고 | 공식 기능 목록 — "TGZ/TBZ 한 단계 해제". tar 묶음 압축을 한 번에 최종 파일까지 푸는 동작만 참고 |
| 화면 | S2 압축 풀기 (+ S7 우클릭 풀기 경로) |
| GUI 요소 | S2-08, S2-04, S2-05, S2-12, S7-01, S7-02, S7-10, S4-01, S4-02, S8-02, S8-11 |
| 디자인 파일 | design/project/Extract.dc.html — 옵션 영역 체크박스 ".tar.gz 를 한 번에 풀기" (켜진 상태로 그려짐) |
| 관련 기능 | #13, #8, #35 |

## 1. 목적

리눅스·오픈소스 배포물은 대부분 `.tar.gz`, `.tar.xz` 형태다. 7-Zip으로 풀면 `.tar` 파일 하나가 생기고 그것을 다시 풀어야 하며, 큰 `.tar`가 디스크에 한 번 더 쓰인다. j-zip은 한 번의 풀기로 tar 안의 실제 파일까지 풀어 준다.

## 2. 현재 7-Zip 동작

- 중첩 자동 열기는 `CPP/7zip/UI/Common/OpenArchive.cpp:3221` `CArchiveLink::Open`에서 함. 바깥 핸들러가 `kpidMainSubfile`을 돌려줄 때만 안쪽을 엶 (`:3309`).
- Gz/Bz2/Zstd 핸들러는 `kpidMainSubfile`을 내지 않음. `XzHandler.cpp:383`은 빈 case. 이 속성을 쓰는 핸들러는 Apm/Ar/Avb/Com/Dmg/Gpt/Lp/Lvm/Mbr/Pe/Qcow/Rpm/Sparse/Split/Vdi/Vhd/Vhdx/Vmdk/Xar뿐.
- CLI `-t` 타입 체인(`ParseOpenTypes`, `OpenArchive.cpp:3655`)도 `kpidMainSubfile`이 있어야 열림.
- FM에는 수동 "Open Inside *"/"#"만 있음 (`FileManager/MyLoadMenu.cpp:752-753`, `IDM_OPEN_INSIDE_ONE`/`IDM_OPEN_INSIDE_PARSER`).
- 컨텍스트 메뉴 풀기 경로: `UI/Common/CompressCall.cpp:255` `ExtractArchives` → 7zG `x`.
- 사용자 입장: `src.tar.gz`를 "여기에 풀기" → `src.tar` 1개 생성. 다시 풀어야 `src\…`가 나옴.

## 3. 요구사항

- FR-12-1 바깥이 gzip·bzip2·xz·zstd 단일 스트림이고 안쪽 단일 항목이 tar이면, S2-08을 표시한다. 판별은 안쪽 데이터가 tar인지로 하며, 확장자(`.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz`, `.tbz2`, `.tar.xz`, `.txz`, `.tar.zst`, `.tzst`)는 표시 여부를 빨리 정하는 보조 수단이다.
- FR-12-2 S2-08이 켜진 상태로 풀면 tar 안의 파일·폴더가 바로 풀리고, 중간 `.tar` 파일은 풀 위치에 생기지 않는다.
- FR-12-3 중간 `.tar`를 임시 파일로 쓰지 않고 스트림으로 이어서 푸는 것을 원칙으로 한다(디스크에 tar 크기만큼 추가로 쓰지 않음). 예외는 §9.
- FR-12-4 S2-08 기본값은 켜짐이다. 끄면 7-Zip과 같이 `.tar` 하나만 나온다.
- FR-12-5 S2-04 폴더 만들기(스마트/항상/없이)와 S2-05 결과 미리보기는 tar 안의 최상위 항목 기준으로 계산한다. 폴더 이름은 `.tar.gz` 등 이중 확장자를 모두 뺀 이름이다(예: `src-1.2.tar.gz` → `src-1.2`).
- FR-12-6 S7-01 "여기에 풀기 (스마트)", S7-02 "\"{이름}\"에 풀기", S7-10 "각각 폴더로 풀기"도 S8-11 기본값에 따라 한 번에 푼다.
- FR-12-7 S4 진행창은 작업 1개로 보이고, 진행률은 바깥 압축 파일을 읽은 비율로 계산한다.
- FR-12-8 S1에서 `.tar.gz`를 열 때 탐색 동작은 바꾸지 않는다(안쪽 `.tar` 1개가 보이고 더블클릭하면 들어감). 자동으로 tar 내용을 보여 줄지는 §9.
- FR-12-9 tar 단계에서 오류(손상, 데이터 끝남)가 나면 S9에 tar 안 파일 단위로 표시한다.

## 4. GUI 명세

| ID | 상태·표시 조건 | 문구 | 동작 |
|---|---|---|---|
| S2-08 | FR-12-1 조건일 때만 표시. 기본 켜짐 | ".tar.gz 등을 한 번에 풀기" (바깥 형식과 관계없이 같은 문구) | 변경 시 S2-05 즉시 갱신 |
| S2-04 | 항상 | 스마트 / 항상 이름 폴더 / 폴더 없이 | tar 안 최상위 항목 기준 |
| S2-05 | 항상 | 예: "D:\Downloads\src-1.2\ ← 맨 위 항목 12개를 묶음" | S2-08 끄면 "D:\Downloads\src-1.2.tar" |
| S2-12 | 항상 | "풀기" | S4로 진행 |
| S7-01 / S7-02 | `.tar.gz` 등 우클릭 | "여기에 풀기 (스마트)" / "\"src-1.2\"에 풀기" | S8 기본값으로 한 번에 풀기 |
| S4-01 / S4-02 | 풀기 중 | "압축 풀기 중 — src-1.2.tar.gz" | 작업 1개 |
| S8-11 | 설정 › 압축 풀기 (S8-02 안) | ".tar.gz 등을 한 번에 풀기", 기본 켜짐 | S2-08 기본값이자 S7 경로 동작 |

## 5. 동작 규칙과 예외

- 바깥 스트림 안의 항목이 tar가 아니면(예: `data.gz` 안에 일반 파일) S2-08을 표시하지 않고 기존처럼 그 파일을 푼다.
- 여러 tar 볼륨이 연결된 경우나 tar 안에 또 `.tar.gz`가 있는 경우, 안쪽 아카이브는 다시 풀지 않는다(한 단계만).
- 분할된 `.tar.gz.001`도 첫 조각을 열면 같은 규칙이 적용된다(분할 인식은 #33).
- tar 안의 심볼릭 링크·절대 경로·`..` 경로는 7-Zip의 기존 경로 검사 규칙을 따른다.
- S2-07 "끝나면 압축 파일을 휴지통으로"는 `.tar.gz` 원본을 대상으로 한다(중간 tar는 원래 없음).
- 취소하면 이미 풀린 파일은 남기고(7-Zip과 같음) 중간 임시 데이터는 지운다.

## 6. 설정

| 키 | 기본값 | 위치(S8 탭) | 설명 |
|---|---|---|---|
| `Extract.TarOneStep` | 켬 | 압축 풀기 (S8-11) | S2-08 기본값이자 S7 경로(대화상자 없이 풀기)의 동작 |

## 7. 구현 지점

research 자료의 두 가지 방식 중 하나를 고른다.

- (a) `Archive/GzHandler.cpp`, `Bz2Handler.cpp`, `XzHandler.cpp`(`:383` 빈 case), `ZstdHandler.cpp`의 `GetArchiveProperty`에서 단일 항목이 tar일 때 `kpidMainSubfile = 0`을 돌려줌 → `CArchiveLink::Open`(`OpenArchive.cpp:3221`, `:3309`)이 자동으로 안쪽을 엶. FM 탐색 동작까지 바뀌므로 FR-12-8을 지키려면 옵션으로 켜고 끌 수 있어야 함.
- (b) `CPP/7zip/UI/Common/Extract.cpp` `DecompressArchive`(36행) 또는 `Extract`(288행)에서 결과가 tar 하나면 이어서 풂.
- S2-08 컨트롤은 `JZip.exe`(WinUI 3) S2 화면에 새로 만든다(기존 파일 `CPP/7zip/UI/GUI/ExtractDialog.cpp`/`ExtractDialogRes.h`는 근거로만 사용). 기본값 저장은 `jzip-core` 설정(기존 `UI/Common/ZipRegistry.h` `NExtract::CInfo` 기반, research 자료 #30과 같은 위치).
- 우클릭 경로: `jzip-shell.dll`이 `JZip.exe`에 넘기는 실행 인자에 이 옵션을 반영한다(기존 `UI/Common/CompressCall.cpp:255` `ExtractArchives`의 7zG 명령줄 구성 기반).

## 8. 수용 기준

- [ ] `src-1.2.tar.gz`(최상위 폴더 `src-1.2/` 1개, 파일 200개)를 S7-01 → `src-1.2\` 폴더 1개, 파일 200개, `src-1.2.tar` 없음
- [ ] 최상위 항목 12개인 `files.tgz`를 S7-01 → 스마트 규칙으로 `files\` 폴더가 생기고 그 안에 12개
- [ ] `.tar.bz2`, `.tbz`, `.tar.xz`, `.tar.zst` 샘플 각각 같은 결과
- [ ] S2에서 S2-08 끄고 풀기 → `src-1.2.tar` 1개만 생김(7-Zip과 같음)
- [ ] S2-08 켜고 끌 때 S2-05 미리보기 문구가 즉시 바뀜
- [ ] 2GB `.tar.gz`를 풀 때 풀 위치·`%TEMP%`에 2GB 넘는 `.tar` 임시 파일이 생기지 않음
- [ ] `data.gz`(안에 `data.csv`) → S2-08 표시 안 됨, `data.csv`가 풀림
- [ ] S1에서 `src-1.2.tar.gz` 열기 → 안쪽 `src-1.2.tar` 1개가 보임(탐색 동작 변화 없음)
- [ ] 여러 개(`a.tar.gz`, `b.tar.xz`) 선택 후 S7-10 "각각 폴더로 풀기" → `a\`, `b\`에 각각 tar 내용이 풀림
- [ ] 뒤쪽을 잘라낸 `.tar.gz` → S9에 "데이터 끝남"과 영향받은 파일 표시

## 9. 범위 밖·미결 사항

- 범위 밖: tar.gz 만들기(#5 F05), 중첩 아카이브 재귀 풀기.
- 미결: (a)/(b) 방식 선택. (a)는 스트림으로 바로 이어져 FR-12-3을 지키기 쉽지만 FM 탐색 동작이 바뀐다(research 메모). (b)는 결과 tar를 디스크에 쓴 뒤 다시 풀 가능성이 있어 FR-12-3과 충돌할 수 있다.
- 미결: FR-12-8 대신 S1에서 `.tar.gz`를 열면 바로 tar 내용을 보여 줄지. 06 S1에는 관련 요소가 없다.
- 미결: 진행률 계산(FR-12-7)에서 바깥 크기 기준이 실제 쓰기량과 다를 때 S4-03 남은 시간 표시 방식.
- 해결: 06 S2-08 표시 대상에 `.tbz2`, `.txz`, `.tzst`를 넣었고, 요소 이름을 ".tar.gz 등을 한 번에 풀기", 기본값을 켜짐으로 정했다.
- 불일치: Extract.dc.html은 `사진_2026.zip`을 여는 화면에 S2-08을 함께 그렸으나 06은 tar 계열에서만 표시하도록 정의한다(디자인은 모든 옵션을 한 화면에 보여 준 예시로 본다).
- 06에 반영됨 (S8-11)
