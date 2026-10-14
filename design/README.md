# j-zip GUI 디자인 (Claude Design 프로젝트 폴더)

이 폴더는 Claude Design 캔버스의 프로젝트 원본이다. 캔버스 없이도 사람·에이전트·다른 모델이 읽고 수정할 수 있도록 형식을 여기에 정리한다.
화면별 기능 명세는 [`../docs/04-gui-reference.md`](../docs/04-gui-reference.md), 기능 번호(#1–#36)는 [`../docs/03-feature-benchmark.md`](../docs/03-feature-benchmark.md)를 기준으로 한다.

```
design/
├── README.md              이 문서
└── project/
    ├── canvas.json        캔버스 인덱스 (아트보드 목록·배치·제목)
    └── *.dc.html          아트보드 1개 = 화면 1개 (Design Component)
```

## 아트보드 목록

| 파일 | 화면 | 크기(px) | 비고 |
|---|---|---|---|
| `Main.dc.html` | S1 · 메인 탐색 창 | 1200×760 | 속성 `dark`(다크모드), `showTags`(기능 배지 표시) |
| `FeatureMap.dc.html` | 기능 → 화면 매핑표 | 960×1000 | #1–#36 데이터가 스크립트 안에 있음 |
| `Extract.dc.html` | S2 · 압축 풀기 | 640×580 | |
| `Compress.dc.html` | S3 · 새로 압축 | 720×760 | 인터랙티브: 간단/고급 전환, 형식 카드 선택 |
| `Progress.dc.html` | S4 · 진행 / 완료 알림 | 560×480 | |
| `Encoding.dc.html` | S5 · 파일명 인코딩 | 640×500 | |
| `Password.dc.html` | S6 · 암호 입력 / 관리자 | 880×500 | |
| `ContextMenu.dc.html` | S7 · 탐색기 우클릭 메뉴 | 760×540 | |
| `Settings.dc.html` | S8 · 설정 | 920×620 | |
| `Error.dc.html` | S9 · 테스트 결과 / 분할 파일 | 760×480 | |

화면 안의 분홍 배지(`.tag`, 예: `#17`)는 기능 번호다.

## `canvas.json` 형식

```jsonc
{
  "v": 3,                                  // 형식 버전
  "createdOnFiles": { "at": "...", "v": 1 },
  "title": "7-Zip GUI 레퍼런스",            // 캔버스 제목
  "launch": { "view": "canvas" },          // 처음 열 때 보기 방식
  "pages": [],
  "boards": {                              // 키 = project/ 안의 파일 경로
    "Main.dc.html": {
      "x": 0, "y": 0,                      // 캔버스 위 위치(px)
      "w": 1200, "h": 760,                 // 아트보드 크기(px)
      "title": "S1 · 메인 탐색 창",
      "is_interactive": true               // (선택) 클릭 동작이 있는 아트보드
    }
  },
  "order": ["Main.dc.html", "..."],        // 뒤→앞 순서
  "notes": {                               // 캔버스 위 제목·메모
    "row1": { "x": 0, "y": -280, "text": "1. 메인 창과 기능 인덱스", "kind": "title1", "maxW": 2240 }
  },
  "designSystems": [],
  "attachments": {}
}
```

배치 규칙: 같은 줄 아트보드 사이 80px, 줄 사이 120px 이상, 줄 제목(`title1`)은 줄 위 223px 이상.

## `.dc.html` 형식 (Design Component)

각 파일은 HTML 한 장이며, 마크업 + 템플릿 구멍 + 작은 로직 클래스로 구성된다.

```html
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<title>화면 이름</title>
<script src="./support.js"></script>      <!-- 캔버스 런타임이 제공. 이 줄은 그대로 둔다 -->
</head>
<body>
<x-dc>                                      <!-- 화면 마크업의 루트 -->
<helmet><style>/* 폰트 링크, 공통 클래스 */</style></helmet>
<div style="width: 640px; height: 580px; ...">
  <h1>{{title}}</h1>                        <!-- 값 구멍: renderVals()가 돌려준 값의 점 경로만 가능 -->
  <sc-for list="{{items}}" as="it">         <!-- 반복 -->
    <div>{{it.name}}</div>
  </sc-for>
  <sc-if value="{{showTags}}">              <!-- 조건 -->
    <span class="tag">#17</span>
  </sc-if>
  <button onClick="{{ toAdv }}">고급</button> <!-- 이벤트: renderVals()가 돌려준 함수 -->
</div>
</x-dc>
<script type="text/x-dc" data-dc-script
        data-props='{"dark":{"editor":"boolean","default":false},"$preview":{"width":640,"height":580}}'>
class Component extends DCLogic {           // 일반 JS, import 없음. React 클래스 컴포넌트와 비슷
  constructor(props) { super(props); this.state = { adv: false }; }
  renderVals() {                            // 템플릿에 넘길 값·함수를 반환
    return { title: '...', items: [], showTags: this.props.showTags ?? true,
             toAdv: () => this.setState({ adv: true }) };
  }
}
</script>
</body>
</html>
```

규칙 요약:

- `{{ }}` 안은 **점 경로만** 가능하다(`{{ a.b }}`). 계산식(`{{ a + b }}`, `{{ !x }}`)은 안 되므로 `renderVals()`에서 계산해 이름으로 넘긴다.
- 스타일은 대부분 인라인 `style="..."`이다. 반복되는 기본 스타일만 `<helmet><style>`의 클래스(`.tag`, `.tb`, `.ic` 등)로 둔다.
- `data-props`는 캔버스 편집기의 조절 항목(예: `dark`, `showTags`)과 미리보기 크기(`$preview`)를 선언한다.
- 모든 요소는 닫는 태그를 쓰고, 속성은 따옴표로 감싼다.
- 외부 리소스는 Google Fonts(Noto Sans KR)만 쓴다. 아이콘은 인라인 SVG다.

## 보는 방법

1. **Claude Design 캔버스로 보기 (권장)**: 새 Design 캔버스를 만들고 `project/` 폴더의 파일들을 같은 경로로 올리면 그대로 열린다. 원본 캔버스(비공개): https://claude.ai/artifact/YXWMXQRdYEgcDggGNiRicM
2. **텍스트로 읽기**: 런타임(`support.js`)이 없으면 브라우저에서 단독으로 열어도 `{{ }}`와 `sc-for`가 처리되지 않는다. 대신 마크업과 `renderVals()`의 데이터를 직접 읽으면 화면 구성을 알 수 있다. 정적 텍스트(라벨·버튼·옵션)는 모두 마크업에 그대로 들어 있다.

## 수정 규칙 (사람·에이전트 공통)

- 화면을 추가하면 `project/<이름>.dc.html`을 만들고 `canvas.json`의 `boards`와 `order`에 같은 경로를 추가한다.
- 화면 크기를 바꾸면 루트 `<div>`의 `width/height`, `data-props`의 `$preview`, `canvas.json`의 `w/h` **세 곳을 함께** 맞춘다.
- 기능 번호 배지를 추가·변경하면 `FeatureMap.dc.html`의 데이터와 `docs/03-feature-benchmark.md`도 같이 고친다.
- 디자인 토큰(색·모서리·간격)은 `docs/04-gui-reference.md`의 "디자인 토큰" 표를 따른다.
- 반디집의 화면·아이콘·브랜딩을 복제하지 않는다. 화면 구성 패턴만 참고한다.
