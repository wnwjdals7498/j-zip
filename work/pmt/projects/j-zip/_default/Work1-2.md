---
type: item
kind: job
id: j-zip/_default/Work1-2
parent: j-zip/_default/Work1
status: Planned
updated: 2026-11-05
blocked_by: []
base_commit: -
---
# [job] 기능 명세 미결 74건 분류와 결정 경로 연결
## 기본 내용
- docs/05-feature-specs/*.md 9절 "- 미결" 항목 74건(2026-11-05 기준 grep 집계). 많은 순: F17 5, F27 4, F13·F21·F24·F26·F32·F36 각 3.
- 미결을 그대로 두면 카드 작성 때 구현자 추측이 생긴다(work/tasks/README.md 5절 2·6항).
## 범위
- [ ] 74건을 세 종류로 분류: (a) 카드 작성 때 카드 결정으로 확정 (b) ADR(work/decisions/) 필요 (c) 시제품·구현 중 실측 필요
- [ ] 각 건에 소유 작업 ID(docs/11) 연결, 소유 작업이 없으면 11에 추가 요청
- [ ] G0–G2 작업 착수를 막는 건은 해당 Item의 blocked_by 또는 PMT backlog로 등록
## 완료 기준
- 74건 전부 분류·소유 작업이 적힌 표가 docs/05-feature-specs/README.md 또는 새 research 문서에 있다
- G0–G2를 막는 건이 PMT에서 blocked_by 또는 backlog 행으로 조회된다(pmt find)
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
