---
type: item
kind: job
id: j-zip/_default/Work1-3
parent: j-zip/_default/Work1
status: In Progress
updated: 2026-11-06
blocked_by: []
base_commit: 80b0494
---
# [job] 초안 작업 카드 4장 검수(작성자와 다른 세션)
## 기본 내용
- work/tasks/ 의 초안 카드 4장: G0-INFRA-T01, G2-F17-T03, G2-F17-T08, G2-F17-T09. 모두 상태 초안.
- 카드 작성자는 자기 카드를 준비됨으로 올리지 않는다(work/tasks/README.md 1·7절). 검수는 작성 세션과 다른 세션이 한다.
## 범위
- [ ] 카드별 work/tasks/README.md 9절 검사 3종(금지어, 섹션 0–14 순서, 기준 커밋 표기)과 사람 확인 6항목 실행
- [ ] 문제는 카드 개정 이력으로 반영 요청, 통과는 검수 결과로 기록
- [ ] 준비됨 전이는 선행 작업 완료 뒤(7절 조건). 선행이 남은 카드는 검수 통과만 기록
## 완료 기준
- 카드 4장 모두 9절 검사 결과(명령·출력)가 이 Item ## 검증 또는 ## 증거에 있다
- 지적 사항이 0건이거나 카드 개정 이력에 반영됐다
## 재개
- 갱신: 2026-11-06T16:44 (session cc-9601c778)
- 한 것: 카드 4장 9절 기계 검사 통과, codex 검수 4회: 지적 10건 중 반영 8, 11 추가 요청 2(B1·B2), T08 kEpsilon 이견 1. 개정 T01 r2·T03 r2·T08 r1·T09 r0.1은 worktree ../j-zip-wt/pmt-Work1-3(docs/pmt-Work1-3-card-review)에 미커밋
- 다음: 사용자 결정: T08 kEpsilon(2순위 이후 동점 판정), 브랜치 커밋·PR 승인. 병합 뒤 end --done
- 주의: 없음
- 대기: 없음
- 검증 못 한 것: 없음
## 검증
| at | commit | criteria | command | exit | limits |
|---|---|---|---|---|---|
| 2026-11-06T16:44 | b91c5de | a58c3ca1 | cd /home/jjm/github/j-zip-wt/pmt-Work1-3 && ! grep -qE '적절히\|필요시\|필요하면\|가능하면\|알아서\|충분히\|기타\|추후\|TODO\|TBD' work/tasks/G*.md && for c in G0-INFRA-T01 G2-F17-T03 G2-F17-T08 G2-F17-T09; do s=$(awk '/^```/{f=!f;next} !f && match($0,/^## ([0-9]+)\\. /,m){printf "%s%s",(n++?",":""),m[1]}' work/tasks/$c.md); [ "$s" = 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14 ] && grep -q '^\| 기준 커밋 \| —(핸드오프 시 채움) \|' work/tasks/$c.md \|\| exit 1; done | 0 | README 9절 기계 검사 3종(PowerShell 대신 bash 동등 명령). 카드 개정은 worktree docs/pmt-Work1-3-card-review에 미커밋. 사람 확인 6항목은 codex gpt-6.1-terra 4회 검수(REVIEW4), T08 kEpsilon 이견 1건 사용자 결정 대기 |
## 결과
-
## 증거
-
