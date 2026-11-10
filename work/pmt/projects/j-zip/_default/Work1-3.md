---
type: item
kind: job
id: j-zip/_default/Work1-3
parent: j-zip/_default/Work1
status: In Progress
updated: 2026-11-10
blocked_by: []
base_commit: bacef2e
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
- 갱신: 2026-11-10T17:40 (session cc-3cc3558d)
- 한 것: T08 kEpsilon 이견을 C안(ADR-0001)으로 해소: F17 FR-17-10~12 보충(agy flash), T08 r2. 브랜치 docs/pmt-Work1-3-card-review를 origin/main 위로 rebase, 커밋 c289b85·2b854cc, PR #1 생성
- 다음: PR #1 병합 뒤 end --done
- 주의: 없음
- 대기: 없음
- 검증 못 한 것: 없음
## 검증
| at | commit | criteria | command | exit | limits |
|---|---|---|---|---|---|
| 2026-11-06T16:44 | b91c5de | a58c3ca1 | cd /home/jjm/github/j-zip-wt/pmt-Work1-3 && ! grep -qE '적절히\|필요시\|필요하면\|가능하면\|알아서\|충분히\|기타\|추후\|TODO\|TBD' work/tasks/G*.md && for c in G0-INFRA-T01 G2-F17-T03 G2-F17-T08 G2-F17-T09; do s=$(awk '/^```/{f=!f;next} !f && match($0,/^## ([0-9]+)\\. /,m){printf "%s%s",(n++?",":""),m[1]}' work/tasks/$c.md); [ "$s" = 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14 ] && grep -q '^\| 기준 커밋 \| —(핸드오프 시 채움) \|' work/tasks/$c.md \|\| exit 1; done | 0 | README 9절 기계 검사 3종(PowerShell 대신 bash 동등 명령). 카드 개정은 worktree docs/pmt-Work1-3-card-review에 미커밋. 사람 확인 6항목은 codex gpt-6.1-terra 4회 검수(REVIEW4), T08 kEpsilon 이견 1건 사용자 결정 대기 |
| 2026-11-10T17:40 | 2b854cc | a58c3ca1 | bash /data/home/jjm/tmp/claude-1000/-data-workspace-github-j-zip/3cc3558d-cb54-4e95-b2c3-2c60885e2037/scratchpad/cardcheck.sh /home/jjm/github/j-zip-wt/pmt-Work1-3 | 0 | README 9절 기계 검사 3종 bash 동등 명령, ADR 금지어 포함. 검사 스크립트는 세션 scratchpad. 문서만 변경이라 빌드·테스트 대상 없음. PR #1 |
## 결과
-
## 증거
-
