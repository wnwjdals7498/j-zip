---
type: item
kind: job
id: j-zip/G5/Work1-5
parent: j-zip/G5/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G4/Work1-1, j-zip/G3/Work1-20]
base_commit: -
---
# [job] G5-F21-T01 악성코드 검사(AMSI·`IAttachmentExecute`) + S8-12·S8-13(MOTW) + S9…
## 기본 내용
- 11 정의: 악성코드 검사(AMSI·`IAttachmentExecute`) + S8-12·S8-13(MOTW) + S9-03 + S4 검사 단계
- 근거: F21
- 크기: L · 필수 증거: E4, E5, E6 + E9
- 선행(11): G4-INFRA-T01, G3-F20-T01
- 원천: docs/11-implementation-backlog.md G5-F21-T01 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 크기 L: 카드 작성 전 PR 하나 크기(400줄 이하)로 나누고 11에 새 작업 ID 추가
- [ ] 작업 카드 work/tasks/G5-F21-T01.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G5-F21-T01/EVIDENCE.md 에 E4, E5, E6 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
