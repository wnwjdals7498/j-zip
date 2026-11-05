---
type: item
kind: job
id: j-zip/_default/Work1-1
parent: j-zip/_default/Work1
status: Done
updated: 2026-11-05
blocked_by: []
base_commit: -
---
# [job] PMT 레포 내 운영 체계 구축과 11번 백로그 등록
## 기본 내용
- PMT 기록을 ~/docs가 아니라 레포 안 work/pmt/ 에 둔다(사용자 결정 2026-11-05). docs/11 백로그 113개를 PMT Item으로 옮겨 진행 상태·재개·차단을 PMT로 추적한다.
## 범위
- [ ] work/pmt/ 에 PMT 프로젝트 j-zip 생성, project.md Goal·Non-Goal·repositories 채움
- [ ] docs/11 작업 113개를 단계별 Work(분류 G0–G6)·Item으로 등록, 11의 선행 열을 blocked_by로 옮김
- [ ] 기획 정리 후속 Item 등록(미결 분류, 카드 검수, PMT 커밋 경로 결정)
- [ ] 운영 규칙 work/pmt/README.md, AGENTS.md·CLAUDE.md, .gitignore(lock 제외)
- [ ] README.md 문서 표·작업 수(113)·상태, docs/09 2절 구조, docs/12 1절 오케스트레이터 쓰기 경로 반영
## 완료 기준
- pmt doctor 결과 fail 0
- 분류별 Item 수 G0 17, G1 8, G2 24, G3 26, G4 21, G5 12, G6 5(합계 113), 모든 Item 제목이 11 작업 ID로 시작
- 검사 스크립트로 11 선행 열과 Item blocked_by가 모두 일치
- git check-ignore 로 .locks/·.repo-locks/ 경로가 무시됨
## 재개

## 검증
| at | commit | criteria | command | exit | limits |
|---|---|---|---|---|---|
| 2026-11-05T10:41 | 9fbe3a4 | 8df4348e | python3 /home/jjm/.claude/skills/proj-mgmt-tool/scripts/pmt.py --docs-root /home/jjm/github/j-zip/work/pmt --project j-zip doctor && python3 /tmp/claude-1000/-home-jjm-github-j-zip/9601c778-8e55-4c16-8128-4b53f20270e6/scratchpad/register_backlog.py --check && git check-ignore -q work/pmt/projects/j-zip/.locks/x work/pmt/.repo-locks/x | 1 | 11↔PMT 검사 스크립트는 세션 scratchpad에 있어 재실행 불가. 변경은 아직 커밋 전(HEAD 9fbe3a4 기준) |
| 2026-11-05T10:41 | 9fbe3a4 | 8df4348e | python3 /home/jjm/.claude/skills/proj-mgmt-tool/scripts/pmt.py --docs-root /home/jjm/github/j-zip/work/pmt --project j-zip doctor && python3 /tmp/claude-1000/-home-jjm-github-j-zip/9601c778-8e55-4c16-8128-4b53f20270e6/scratchpad/register_backlog.py --check && git check-ignore -q work/pmt/projects/j-zip/.locks/x && git check-ignore -q work/pmt/.repo-locks/x | 0 | 11↔PMT 검사 스크립트는 세션 scratchpad에 있어 재실행 불가. 변경은 아직 커밋 전(HEAD 9fbe3a4 기준) |
## 결과
- PMT를 레포 안 work/pmt/로 옮겨 운영 시작. docs/11 작업 113개를 G0–G6 Item으로 등록(선행 259건→blocked_by), 후속 Item 3건, 운영 규칙 work/pmt/README.md·AGENTS.md·CLAUDE.md 추가, README·docs/09·docs/12·.gitignore 반영
## 증거
- work/pmt/README.md
