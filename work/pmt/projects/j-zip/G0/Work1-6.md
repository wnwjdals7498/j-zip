---
type: item
kind: job
id: j-zip/G0/Work1-6
parent: j-zip/G0/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G0/Work1-3]
base_commit: -
---
# [job] G0-INFRA-T05 코퍼스 v0 생성기 `tests/corpus/generate.ps1` + 통합 테스트 실행기…
## 기본 내용
- 11 정의: 코퍼스 v0 생성기 `tests/corpus/generate.ps1` + 통합 테스트 실행기 `tests/integration/run.ps1`, 원본 `7z.exe` 기준선 통과. 실행기는 로직 묶음도 돌린다: `-Suite <이름>`이 7z.exe 코퍼스 묶음 또는 통합 테스트 실행 파일의 로직 이름 묶음(`tests/corpus/<이름>/`) 중 하나를 고른다(09 11절)
- 근거: 09 3.2 #1·#6, 09 11절
- 크기: M · 필수 증거: E1, E3 + E9
- 선행(11): T02
- 원천: docs/11-implementation-backlog.md G0-INFRA-T05 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G0-INFRA-T05.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G0-INFRA-T05/EVIDENCE.md 에 E1, E3 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
