---
type: item
kind: job
id: j-zip/_default/Work1-4
parent: j-zip/_default/Work1
status: Done
updated: 2026-11-05
blocked_by: []
base_commit: 9fbe3a4
---
# [job] PMT 기록 커밋 경로 결정(G0 착수 전)
## 기본 내용
- PMT 기록(work/pmt/)을 레포 안에 두기로 했다(사용자 결정 2026-11-05). 구현 단계에서는 작업마다 브랜치·worktree·PR을 쓰고 main 직접 push를 금지한다(docs/09 7절). PMT 상태는 작업을 가로지르므로 브랜치마다 따로 두면 충돌한다.
- 정해야 할 것: PMT 변경을 어느 브랜치에 어떤 커밋으로 남기는지, CI 기록 전용 커밋 정의(docs/09 1·7절, docs/10 3.3, docs/12 4.5, docs/11 G0-INFRA-T03)에 work/pmt/** 를 넣을지.
## 범위
- [ ] 선택지 3개 이상 비교(장점·단점·적합 조건) 후 사용자 결정
- [ ] 결정을 pmt decide 로 기록하고 work/pmt/README.md 와 관련 문서 반영
## 완료 기준
- decisions.md 에 승인 결정 행이 있다
- work/pmt/README.md 커밋 규칙 절이 결정과 일치한다
- G0-INFRA-T16(j-zip/G0/Work1-1)의 blocked_by 에서 이 Item이 풀렸다
## 재개

## 검증
| at | commit | criteria | command | exit | limits |
|---|---|---|---|---|---|
| 2026-11-05T10:52 | 93a1291 | b85c3126 | grep -q '^\| D1 \| 승인' work/pmt/projects/j-zip/decisions.md && grep -q 'PMT 결정 D1' work/pmt/README.md && test -z "$(grep -rnE 'work/handoff/\\*\\*' docs work/tasks \| grep -E '기록 전용 커밋\|경로 필터\|줄 수에서' \| grep -v 'work/pmt' \| grep -vE '12-agent-orchestration.md:(233\|719):')" | 0 | 719행(12의 과거 반영 이력)·233행(.gitignore 설명)은 정의가 아니라 제외 |
## 결과
- D1 승인(A안): PMT 변경은 main에 기록 전용 커밋으로 직접 커밋·push. 기록 전용 정의·CI 경로 필터·줄 수 제외에 work/pmt/** 추가, main 직접 push 예외로 명시(커밋 93a1291)
## 증거
- work/pmt/projects/j-zip/decisions/D1.md
