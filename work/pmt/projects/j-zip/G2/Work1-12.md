---
type: item
kind: job
id: j-zip/G2/Work1-12
parent: j-zip/G2/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G0/Work1-12]
base_commit: -
---
# [job] G2-F17-T09 상용 문자 빈도표 ADR과 표 생성
## 기본 내용
- 11 정의: 상용 문자 빈도표 ADR과 표 생성: 한글 상위 512자·간체 한자 상위 512자의 출처·라이선스 결정(ADR), 생성 스크립트, `jzip-shared`의 생성된 표 파일. 허용 라이선스 원본이 없으면 보류(사용자 결정)
- 근거: F17 FR-17-8
- 크기: M · 필수 증거: E1, E2 + ADR + E9
- 선행(11): G0-INFRA-T11
- 원천: docs/11-implementation-backlog.md G2-F17-T09 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G2-F17-T09.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G2-F17-T09/EVIDENCE.md 에 E1, E2 + ADR + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
