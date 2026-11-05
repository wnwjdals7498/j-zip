# j-zip 에이전트 규칙

이 파일은 이 레포에서 일하는 모든 코딩 에이전트(Claude Code, Codex, Antigravity)가 먼저 읽는 규칙이다. 세부 규칙은 아래에 링크한 문서가 원천이다.

## 1. 레포 안에서 완결

- 이 레포의 작업은 이 레포 안의 문서와 기록만으로 완결한다. 계획, 진행 상태, 결정, 증거를 레포 밖(`~/docs`, 개인 메모 등)에 원천으로 두지 않는다.
- 문서 지도는 [README.md](README.md)에 있다.

## 2. 진행 관리(PMT)

- 진행 상태, 재개 정보, 차단 관계, 운영 결정은 [work/pmt/](work/pmt/README.md)의 PMT(`proj-mgmt-tool` 스킬)로 관리한다.
- PMT 명령은 반드시 데이터 루트를 이 레포의 `work/pmt/`로 지정해 실행한다. 지정하지 않으면 기본값 `~/docs`에 기록된다. 지정 방법은 [work/pmt/README.md](work/pmt/README.md) 2절을 따른다.
- 작업을 시작하거나 이어갈 때는 `resume j-zip` 출력과 작업할 Item 하나만 읽는다.

## 3. 역할과 절차

- 역할(오케스트레이터, 구현, 검토, 검증)과 쓸 수 있는 경로는 [docs/12](docs/12-agent-orchestration.md) 2절을 따른다. `work/pmt/**`는 오케스트레이터만 쓴다.
- 단계(G0–G6), 브랜치·PR, 완료 정의는 [docs/09](docs/09-development-process.md), 작업 목록은 [docs/11](docs/11-implementation-backlog.md), 작업 카드는 [work/tasks/README.md](work/tasks/README.md)를 따른다.
