---
type: item
kind: job
id: j-zip/G0/Work1-8
parent: j-zip/G0/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G0/Work1-4]
base_commit: -
---
# [job] G0-INFRA-T07 증거 도구
## 기본 내용
- 11 정의: 증거 도구: `new-manifest.ps1`(`RUNLOG.md`가 있으면 "외부 실행 기록" 절 생성), CI의 `JZIP-VERIFY` 댓글 반영 봇, PR별 `evidence-pr-<번호>.zip`을 초안 릴리스 `gate-G<n>`에 첨부. 병합 때 `수동 작성(증거 도구 이전)` 매니페스트(T01~T06 등)에 "백필 안내" 행을 한 번 덧붙임(10 3.3)
- 근거: 10 3.3, 09 11절
- 크기: M · 필수 증거: E1 + E9
- 선행(11): T03
- 원천: docs/11-implementation-backlog.md G0-INFRA-T07 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G0-INFRA-T07.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G0-INFRA-T07/EVIDENCE.md 에 E1 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
