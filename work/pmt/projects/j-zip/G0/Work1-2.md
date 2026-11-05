---
type: item
kind: job
id: j-zip/G0/Work1-2
parent: j-zip/G0/Work1
status: Planned
updated: 2026-11-05
blocked_by: [j-zip/G0/Work1-1]
base_commit: -
---
# [job] G0-INFRA-T01 레포 골격
## 기본 내용
- 11 정의: 레포 골격: 폴더 구조(`spikes/`·`tests/scratch/`는 `.gitkeep`), 7-Zip 26.04 subtree(`upstream/7zip`, **오케스트레이터가 핸드오프 전에 가져옴**, 12 4.10), `upstream/PATCHES.md`, `third_party/VERSIONS.md`, `THIRD_PARTY_NOTICES.md` 초안. PR은 병합 커밋으로 병합(09 7절)
- 근거: 09 2절, 09 3.2 #12
- 크기: S · 필수 증거: E1 + E9
- 선행(11): T16
- 원천: docs/11-implementation-backlog.md G0-INFRA-T01 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G0-INFRA-T01.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G0-INFRA-T01/EVIDENCE.md 에 E1 + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
