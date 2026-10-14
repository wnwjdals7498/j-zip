# 05. 기능 세부 명세

[03. 기능 벤치마크](../03-feature-benchmark.md)의 기능 #1–#36을 하나씩 명세한다.
각 문서는 **반디집의 어느 기능을 참고했는지**, **7-Zip 26.04의 현재 동작과 근거**, **GUI 명세서의 어느 화면 요소에 연결되는지**를 함께 적는다.

- 화면 요소 ID(`S2-04` 등): [06. GUI 명세서](../06-gui-spec.md)
- 7-Zip 근거(파일·줄): [research/7zip-26.04-source-facts.md](../research/7zip-26.04-source-facts.md)
- 디자인 원본: [`design/project/`](../../design/project/)

## 목록

| # | 문서 | 순위 | 7-Zip | 화면 |
|---|---|---|---|---|
| 1 | [F01 분할 압축](F01-split-archive.md) | — | ✅ | S3 |
| 2 | [F02 4GB 넘는 아카이브](F02-large-archive.md) | — | ✅ | S3 |
| 3 | [F03 멀티코어 압축](F03-multicore.md) | — | ✅ | S3 |
| 4 | [F04 AES-256 암호](F04-aes256.md) | — | ✅ | S3 S6 |
| 5 | [F05 추가 압축 포맷 생성](F05-create-formats.md) | P3 | △ | S3 |
| 6 | [F06 자동 해제 EXE](F06-sfx.md) | P3 | △ | S3 |
| 7 | [F07 ALZ/EGG 해제](F07-alz-egg.md) | P2 | ❌ | S1 S2 |
| 8 | [F08 ZSTD/LZ4 해제](F08-zstd-lz4.md) | P3 | △ | S1 |
| 9 | [F09 ACE 해제 (제외)](F09-ace.md) | 제외 | ❌ | — |
| 10 | [F10 풀지 않고 목록 보기](F10-browse.md) | — | ✅ | S1 |
| 11 | [F11 드래그해서 꺼내기](F11-drag-out.md) | P2 | △ | S1 |
| 12 | [F12 TGZ/TBZ 한 번에 풀기](F12-tar-one-step.md) | P1 | ❌ | S2 |
| 13 | [F13 스마트 해제](F13-smart-extract.md) | P1 | △ | S2 S7 |
| 14 | [F14 풀지 않고 아카이브 수정](F14-modify-archive.md) | — | ✅ | S1 |
| 15 | [F15 아카이브 안 파일 바로 편집](F15-edit-inside.md) | P2 | △ | S1 |
| 16 | [F16 주석 보기](F16-comment.md) | P3 | △ | S1 |
| 17 | [F17 파일명 인코딩 자동 감지](F17-filename-encoding.md) | P1 | ❌ | S1 S5 |
| 18 | [F18 우클릭 메뉴](F18-context-menu.md) | — | ✅ | S7 |
| 19 | [F19 Win11 새 컨텍스트 메뉴](F19-win11-menu.md) | P2 | ❌ | S7 |
| 20 | [F20 무결성 테스트](F20-integrity-test.md) | — | ✅ | S1 S9 |
| 21 | [F21 악성코드 검사](F21-malware-scan.md) | P3 | ❌ | S8 S9 |
| 22 | [F22 다국어](F22-languages.md) | — | ✅ | S8 |
| 23 | [F23 비밀번호 관리자](F23-password-manager.md) | P2 | ❌ | S6 |
| 24 | [F24 손상 ZIP 복구](F24-zip-repair.md) | P3 | △ | S9 |
| 25 | [F25 비밀번호 찾기 (제외)](F25-password-recovery.md) | 제외 | ❌ | — |
| 26 | [F26 이미지 썸네일](F26-thumbnail.md) | P2 | ❌ | S1 |
| 27 | [F27 ARM64·포터블·스토어 배포](F27-distribution.md) | P3 | △ | 배포 |
| 28 | [F28 다크모드](F28-dark-mode.md) | P2 | ❌ | S1 S8 |
| 29 | [F29 고DPI·모던 아이콘](F29-high-dpi.md) | P2 | △ | 전체 |
| 30 | [F30 해제 후 폴더 열기](F30-open-folder-after.md) | P1 | ❌ | S2 S4 |
| 31 | [F31 해제 후 원본 휴지통](F31-recycle-after.md) | P1 | ❌ | S2 S4 S8 |
| 32 | [F32 자동 닫기·완료 알림](F32-auto-close-notify.md) | P1 | △ | S4 |
| 33 | [F33 분할 파일 자동 인식](F33-multivolume-detect.md) | P2 | △ | S1 S9 |
| 34 | [F34 메뉴 항목 선택](F34-menu-customize.md) | P3 | ✅ | S8 |
| 35 | [F35 여러 개 각각 폴더로 풀기](F35-extract-each.md) | — | ✅ | S7 |
| 36 | [F36 탭 탐색기](F36-tabs.md) | P3 | ❌ | S1 |

## 문서 양식

모든 기능 문서는 아래 양식을 따른다. 이미 있는 기능(✅, 순위 —)과 제외 기능은 해당 없는 절을 "해당 없음"으로 짧게 적는다.

```markdown
# F{NN}. {기능 이름}

| 항목 | 내용 |
|---|---|
| 기능 번호 | #{N} |
| 우선순위 | P1 / P2 / P3 / — / 제외 |
| 7-Zip 26.04 | ✅ / △ / ❌ — 한 줄 요약 |
| 반디집 참고 | 출처 구분(공식 기능 목록 / Pro 기능 / 실사용 UX) + 참고한 동작 한 줄 |
| 화면 | S{n} 화면 이름 |
| GUI 요소 | S{n}-{nn}, … (06. GUI 명세서의 요소 ID) |
| 디자인 파일 | design/project/{파일}.dc.html — 해당 부분 설명 |
| 관련 기능 | #{n} … |

## 1. 목적
사용자 문제와 이 기능이 해결하는 것. 2–4문장.

## 2. 현재 7-Zip 동작
근거 파일·함수·줄(research 자료 기준)과 사용자 입장에서 보이는 동작.

## 3. 요구사항
- FR-{NN}-1 …  (반드시 지켜야 할 동작, 검증 가능하게)

## 4. GUI 명세
요소 ID별 표: ID | 상태·표시 조건 | 문구 | 동작. 06 문서와 어긋나지 않게.

## 5. 동작 규칙과 예외
오류, 경계 조건, 다른 기능과의 상호작용.

## 6. 설정
| 키 | 기본값 | 위치(S8 탭) | 설명 |

## 7. 구현 지점
7-Zip 소스 기준 변경 위치(파일·클래스·함수·리소스 ID). research 자료에 있는 것만.

## 8. 수용 기준
- [ ] 확인 가능한 체크리스트

## 9. 범위 밖·미결 사항
```
