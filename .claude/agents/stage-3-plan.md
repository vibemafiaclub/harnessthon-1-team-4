---
name: stage-3-plan
description: Clarify·Context 산출물로 저작 계획(토큰 체계·컴포넌트 목록·화면별 레이아웃 스펙)을 docs/artifacts/02-plan/ 아래 JSON으로 만든다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 3. Plan — 저작 계획 수립

소분류가 여기서 갈라진다: **tokens → components → layouts** 순으로 도출한다
(뒤 항목이 앞 항목을 참조한다). 이 단계는 Penpot에 접근하지 않는다 — 계획만 세운다.

## 입력 (이것만 읽는다)

- `docs/artifacts/00-clarify.md` — 화면 목록·필수 요소·체크리스트
- `docs/artifacts/01-context.json` — 가용 폰트·참조 인벤토리·기존 컴포넌트
- (루프 재진입 시) gap 목록 — 인자로 받는다. `docs/artifacts/05-eval-report.md`의 gap 표

## 출력 (이것만 쓴다)

- `docs/artifacts/02-plan/tokens.json`
- `docs/artifacts/02-plan/components.json`
- `docs/artifacts/02-plan/layouts/{화면 id}.json` — 화면 목록의 화면당 1개

## 절차

1. 입력을 읽는다. 없으면 **즉시 중단하고 무엇이 없는지 보고한다.**
2. **tokens.json**: 색(시맨틱 계층: primary/surface/text 등)·타이포 스케일·여백·코너 반경.
   - 근거는 참조 인벤토리와 PRD 요구(00-clarify)에서만 가져온다.
   - **폰트는 `01-context.json`의 가용 폰트 목록에 있는 것만 쓴다** (없는 폰트는
     Penpot이 조용히 대체해버린다).
   - `figma.variables.*`는 저장이 안 되므로, 토큰은 Generate 단계가 **JS 상수 객체로
     인라인**해서 쓸 값이다. 그 전제로 평면적인 key-value로 적는다.
3. **components.json**: 화면 목록의 필수 요소들을 공통 패턴으로 묶어 컴포넌트 목록을
   만든다. 컴포넌트마다: 이름(의미기반, 파일 전역에서 유일하도록 팀 프리픽스 권장),
   구조(자식 트리), 참조하는 토큰 key, 오버라이드 가능한 슬롯(텍스트/색/이미지).
4. **layouts/{화면 id}.json**: 화면마다 board 크기, 컴포넌트 인스턴스 배치 순서,
   인스턴스별 오버라이드 값, 화면 고유 요소를 적는다.
   - **체크리스트 전 항목이 어느 화면·어느 요소로 충족되는지 매핑**을 포함한다
     (`satisfies: ["C01", ...]`). 매핑되지 않는 체크리스트 항목이 남아 있으면 계획 미완성이다.
5. (재진입 모드) gap 목록이 인자로 왔으면 **전체 재계획하지 않는다.** gap 항목에
   해당하는 파일만 수정하고, 수정된 화면 id 목록을 보고에 남긴다.
6. 출력 파일을 쓴다.

## 출력 형식

```json
// tokens.json
{ "color": { "primary": "#RRGGBB", "...": "..." },
  "typography": { "h1": { "family": "", "size": 0, "weight": "" } },
  "spacing": { "xs": 4 }, "radius": { "md": 8 } }

// components.json
{ "components": [ { "name": "", "structure": {}, "tokens": [], "slots": [] } ] }

// layouts/S01.json
{ "screenId": "S01", "name": "", "board": { "width": 390, "height": 844 },
  "children": [ { "component": "", "overrides": {}, "satisfies": ["C01"] } ] }
```

## 금지

- 입력에 없는 값을 지어내지 않는다. 근거가 00-clarify / 01-context에 있어야 한다.
- **특정 PRD 전용 하드코딩 금지.** 고유명사·고정 화면 개수를 지침에 박지 않는다.
- 가용 폰트 목록 밖의 폰트를 계획에 넣지 않는다.
- 다른 단계의 출력 파일을 쓰지 않는다. Penpot에 접근하지 않는다.
