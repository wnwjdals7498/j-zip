---
type: item
kind: job
id: j-zip/G0/Work1-1
parent: j-zip/G0/Work1
status: Planned
updated: 2026-11-05
blocked_by: []
base_commit: -
---
# [job] G0-INFRA-T16 핸드오프 경로 스모크 테스트
## 기본 내용
- 11 정의: 핸드오프 경로 스모크 테스트: 버리는 worktree에서 Codex(`gpt-6-luna`)·Antigravity(`gemini-3.8-flash-medium`)를 각각 한 번 실행해 모델 고정, Windows에서의 쓰기 범위 제한, 출력 수집, 시간 초과 처리를 확인. **통과 전에는 다른 외부 핸드오프 금지**
- 근거: [12](12-agent-orchestration.md) 9절, 09 3.2 #11
- 크기: M · 필수 증거: E4 + RUNLOG + ADR + E9
- 선행(11): —
- 원천: docs/11-implementation-backlog.md G0-INFRA-T16 행. 정의가 바뀌면 11을 먼저 고치고 이 Item을 맞춘다
## 범위
- [ ] 작업 카드 work/tasks/G0-INFRA-T16.md 작성과 다른 세션 검수(상태 준비됨)
- [ ] 핸드오프·구현·검토·검증·병합(docs/12 5.1 수명주기)
## 완료 기준
- 카드 상태 완료: reviewer 승인, verifier JZIP-VERIFY result=pass, 병합(work/tasks/README.md 7절)
- 증거 매니페스트 work/evidence/G0-INFRA-T16/EVIDENCE.md 에 E4 + RUNLOG + ADR + E9 기록(docs/10 3.2·3.3)
- docs/09 8절 완료 정의(DoD) 7개 항목 충족
## 재개
<start 이후 note가 채움>
## 결과
-
## 증거
-
