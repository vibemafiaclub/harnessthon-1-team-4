---
name: stage-2-context
description: Penpot 실행 환경(대상 Page·가용 폰트·기존 라이브러리·참조 디자인 인벤토리)을 읽기 전용으로 조사해 docs/artifacts/01-context.json으로 만든다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 2. Context Gather — 실행 환경 조사

이 단계는 **읽기 전용**이다. Penpot에 아무것도 만들지 않고, 아무것도 수정하지 않는다.

## 입력 (이것만 읽는다)

- **작업 Page 이름** — 인자로 받는다. **없으면 조사 없이 즉시 중단하고 요구한다.**
  기본값으로 첫 Page를 쓰지 않는다. 추측하지 않는다.
- 참조(기존 자산) Page 이름 목록 — 인자로 받는다. 기본값: `1-daangn`, `2-airbnb`
- Penpot MCP (`use_figma`) — 읽기 호출만

## 출력 (이것만 쓴다)

- `docs/artifacts/01-context.json`

## 절차

1. Page 인자를 확인한다. 없으면 즉시 중단·보고.
2. `penpot.currentFile.pages`로 전체 Page 목록을 읽고, 작업 Page의 존재를 확인한다.
   없으면 중단·보고 (만들지 않는다 — Page 생성은 운영 소관).
3. **Page를 전환하지 않는다.** 실시간 협업 파일이라 전환하면 남이 보는 화면이 바뀐다.
   읽기는 `pages.find(p => p.name === ...)` 로 Page 객체를 얻어 그 안을 순회한다.
   (실증된 경로: getPageByName → 자식 shape 전수 스캔)
4. `penpot.fonts.all`로 가용 폰트 목록을 얻는다.
   **`findByName`은 유사 매칭이라 쓰지 않는다** ("Inter" → "Inter Tight"가 잡힘).
   반드시 `fonts.all`에서 이름 정확 비교.
5. 파일의 기존 컴포넌트·라이브러리 상태를 조사한다 (이름·id — 이름은 파일 전역이므로
   나중 단계가 id 프리픽스로 좁혀 찾을 수 있게 id를 남긴다).
6. 참조 Page들을 순회해 스타일 인벤토리를 뽑는다: 사용된 색 클러스터, 타이포
   (폰트·크기·굵기) 클러스터, 공통 여백·코너 반경 경향.
7. JSON을 쓴다.

## 출력 형식

```json
{
  "targetPage": { "name": "", "exists": true, "boardCount": 0 },
  "pages": [ { "name": "", "id": "" } ],
  "fonts": [ { "family": "", "weights": [] } ],
  "existingComponents": [ { "name": "", "id": "" } ],
  "referenceInventory": {
    "colors": [ { "hex": "", "usage": "", "count": 0 } ],
    "typography": [ { "family": "", "size": 0, "weight": "", "usage": "" } ],
    "spacing": [], "radius": []
  }
}
```

## 금지

- **Page 전환(`openPage`) 금지.** 쓰기·수정 호출 금지. 참조 Page는 수정 금지.
- 입력에 없는 값을 지어내지 않는다. 조사 실패 항목은 `null` + `errors` 배열에 사유를 적는다.
- 특정 PRD 전용 하드코딩 금지. 다른 단계의 출력 파일을 쓰지 않는다.
