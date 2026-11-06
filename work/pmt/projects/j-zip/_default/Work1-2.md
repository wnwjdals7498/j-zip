---
type: item
kind: job
id: j-zip/_default/Work1-2
parent: j-zip/_default/Work1
status: In Progress
updated: 2026-11-06
blocked_by: []
base_commit: 80b0494
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
- 갱신: 2026-11-06T16:44 (session cc-9601c778)
- 한 것: 미결 74줄 분류표 작성(agy flash), codex 독립 검토 반영: a19 b14 c37 제외4. G0-G2 차단 5건은 backlog B3-B5, 11 추가 요청 B6. 분류표는 worktree ../j-zip-wt/pmt-Work1-2(docs/pmt-Work1-2-open-items-triage)에 미커밋
- 다음: 브랜치 커밋·PR 승인 뒤 병합, Item 기본 내용 74건→실제 70건 정정, end --done
- 주의: 없음
- 대기: 없음
- 검증 못 한 것: 없음
## 검증
| at | commit | criteria | command | exit | limits |
|---|---|---|---|---|---|
| 2026-11-06T16:44 | b91c5de | 7b0800e8 | python3 /tmp/claude-1000/-home-jjm-github-j-zip/9601c778-8e55-4c16-8128-4b53f20270e6/scratchpad/check_triage.py /home/jjm/github/j-zip-wt/pmt-Work1-2/docs/research/open-items-triage-2026-11.md '^(F\\d\\d\|OPS):' \| grep -qx 'rows 74 bad 1 dups 0' | 0 | 검사 스크립트는 세션 scratchpad에만 있다. bad 1은 64번 근거가 다른 작업 행(11:116)을 의도적으로 인용한 것. 분류표는 worktree docs/pmt-Work1-2-open-items-triage에 미커밋. 분류 판정은 codex gpt-6.1-terra가 독립 검토(22행 지적 중 21행 반영, 25번 차단은 a 분류라 유지) |
## 결과
-
## 증거
-
