---
type: item
kind: job
id: j-zip/G2/Work1-3
parent: j-zip/G2/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G2/Work1-2]
base_commit: -
---
# [job] G2-INFRA-T03 설정 서비스(`jzip-core`, 주입된 폴더) + 쓰기 규칙(뮤텍스·병합·원자적 교체·재시도…
## 기본 내용
- 11 정의: 설정 서비스(`jzip-core`, 주입된 폴더) + 쓰기 규칙(뮤텍스·병합·원자적 교체·재시도·`Result`) + `jzip-win` 공유 읽기 + `ZipRegistry` 어댑터 패치(`JZIP-PATCH`)
- 근거: 08 6절
- 크기: L · 필수 증거: E1, E2 + E9
- 선행(11): G2-INFRA-T02
- 원천: docs/11-implementation-backlog.md G2-INFRA-T03 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 크기 L: 카드 작성 전 PR 하나 크기(400줄 이하)로 나누고 11에 새 작업 ID 추가
- [ ] 작업 카드 work/tasks/G2-INFRA-T03.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G2-INFRA-T03/EVIDENCE.md 에 E1, E2 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
