---
type: item
kind: job
id: j-zip/G0/Work1-12
parent: j-zip/G0/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G0/Work1-3]
base_commit: -
---
# [job] G0-INFRA-T11 `jzip-shared`·`jzip-win` 빈 라이브러리 두 변형(`-mt`·`-md`)
## 기본 내용
- 11 정의: `jzip-shared`·`jzip-win` 빈 라이브러리 두 변형(`-mt`·`-md`), nmake 변수 `JZIP_SHARED_LIB`·`JZIP_WIN_LIB`로 `7z.dll`·`7z.exe` 링크, 플래그 계약
- 근거: 09 3.2 #2·#3, 07 5절
- 크기: M · 필수 증거: E1 + E9
- 선행(11): T02
- 원천: docs/11-implementation-backlog.md G0-INFRA-T11 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G0-INFRA-T11.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G0-INFRA-T11/EVIDENCE.md 에 E1 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
