---
type: item
kind: job
id: j-zip/G2/Work1-8
parent: j-zip/G2/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G2/Work1-11]
base_commit: -
---
# [job] G2-F17-T04 `ZipItem`·`ZipHandler` 연결 패치
## 기본 내용
- 11 정의: `ZipItem`·`ZipHandler` 연결 패치: 감지 입력 선택(FR-17-3 처리량 상한 이름 2,000개·256KiB 포함), 이름 결정 순서(FR-17-13), Unix 규칙 처리, 항목별 UTF-8 예외, `cp` 속성
- 근거: F17 FR-17-3, FR-17-13~16
- 크기: M · 필수 증거: E1, E2, E3 + E9
- 선행(11): G2-F17-T08
- 원천: docs/11-implementation-backlog.md G2-F17-T04 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G2-F17-T04.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G2-F17-T04/EVIDENCE.md 에 E1, E2, E3 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
