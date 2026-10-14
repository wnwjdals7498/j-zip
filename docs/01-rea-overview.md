# 01. REA 개요

- 저장소: https://github.com/morluto/rea
- 한 줄 요약: "Reverse engineer anything with agents" — AI 에이전트가 소프트웨어를 분석하도록 돕는 오픈소스 **MCP 서버**
- 용도 범위: 합법적인 연구·교육·상호운용성 확보, 본인 소유이거나 분석 권한이 있는 소프트웨어

## 주요 기능

- 네이티브 바이너리 분석: Hopper / Ghidra / IDA 연동 (의사코드, 어셈블리, 문자열, 심볼, 참조)
- JS / Electron 앱 분석: 모듈, IPC, 라우트, 네이티브 애드온 (네이티브 엔진 불필요)
- .NET, Android APK, Flutter, WASM, EVM 바이트코드, SQLite, 펌웨어 검사
- 웹사이트·네트워크 캡처(pcap) 분석, 프로세스 동작 캡처
- adb를 통한 Android 기기 연동
- MCP 도구와 같은 기능을 CLI로 제공

## 기술 스택

- TypeScript / Node.js, Vitest, Stryker(뮤테이션 테스트), oxlint/oxfmt
- 외부 연동: Hopper, Ghidra, IDA, JADX, Rizin, TShark, pwntools
- 지원 에이전트: Claude Code, Codex, Cursor, Gemini CLI

## 설치

```bash
npx rea-agents setup                      # 에이전트에 MCP 등록 후 에이전트 재시작
npm install --global rea-agents && rea --help   # CLI 사용
```

- Node.js 22.x(≥22.19), 24.x(≥24.11), 26 이상
- 라이선스: REA 코드는 MIT. IDA, Hopper, JEB 등 상용 도구는 별도로 합법 취득 필요
- **공식 출처만 사용**: morluto/rea 저장소, rea.tools, npm `rea-agents`. 이름이 비슷한 가짜 패키지 주의

## j-zip에서의 역할

7-Zip은 소스가 공개되어 있으므로 **주 작업은 소스 읽기·수정**이다. REA는 보조 도구로만 쓴다.

- 수정 전/후 빌드의 **프로세스 동작 캡처**를 비교해 파일 쓰기 경로, 레지스트리 변경을 검증 (특히 셸 확장)
- 직접 빌드한 바이너리가 의도대로 동작하는지 확인
- 상용 압축 프로그램(반디집, 알집 등)의 바이너리 분석에는 사용하지 않는다
