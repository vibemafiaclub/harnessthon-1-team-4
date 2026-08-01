---
name: stage-5-generate-screens
description: Plan의 레이아웃 스펙과 라이브러리 로그로 화면 board를 조립하고, 화면→노드 ID 매핑을 docs/artifacts/04-screens-log.json으로 남긴다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 5. Generate: Screens — 화면 조립

Penpot **쓰기 단계**다. stage-4(라이브러리)가 끝난 뒤에만 실행한다 (Penpot 쓰기 직렬).

## 🔴 작업 Page 게이트 (저작 전 필수)

- **Page 이름을 인자로 받는다. 받지 못했으면 저작을 시작하지 않고 사용자에게 묻는다.**
- 기본값으로 첫 Page를 쓰지 않는다. 추측해서 고르지 않는다.
- `중간공유`·`최종제출`은 공용 Page다. 여기서 처음부터 저작하지 않는다 (옮겨 담는 곳).
- 기존 자산 Page(`1-daangn`, `2-airbnb` 등 참조 Page)는 **읽기 전용. 수정 금지.**

## 입력 (이것만 읽는다)

- `docs/artifacts/02-plan/layouts/*.json` — 화면별 스펙
- `docs/artifacts/03-library-log.json` — 컴포넌트 id 매핑
- 작업 Page 이름 (인자)
- (루프 재진입 시) gap 목록 — 인자. 대상 노드 ID가 함께 온다

## 출력 (이것만 쓴다)

- Penpot 작업 Page의 화면 board들 (부수 효과)
- `docs/artifacts/04-screens-log.json`

## 절차

1. 입력을 읽는다. 없으면 **즉시 중단하고 무엇이 없는지 보고한다.**
   layouts의 화면 수와 실제 파일 수가 다르면 그것도 중단 사유다.
2. Page 전환은 **별도 호출로 먼저**, 이후 **모든 스크립트 첫 줄에서 Page 재고정**.
3. 화면을 하나씩 조립한다. **작게 쪼개 실행 + 검증** 반복:
   - 컴포넌트는 `03-library-log.json`의 **id로 찾아** 인스턴스를 만든다
     (이름 검색은 파일 전역이라 남의 컴포넌트가 잡힌다).
   - 데이터만 다른 반복 행은 **인스턴스 재사용**: `characters` 오버라이드는 되고,
     색은 penpot 형식 fills(`{fillColor, fillOpacity}`)로 오버라이드한다.
   - 같은 화면의 상태 변형(모달·에러·로딩·빈 상태)은 **`shape.clone()`** 후 덮을 것만
     얹는다. 처음부터 다시 짓지 않는다.
   - 실제 사진이 필요하면 `await penpot.uploadMediaUrl(name, url)` →
     `fills = [{ fillOpacity:1, fillImage: img }]`.
   - 반투명 스크림은 렌더링에서 사라질 수 있다 → 뒤 화면 board의 `opacity`를 낮춘다.
   - 비-오토레이아웃 프레임에 `appendChild`한 자식은 좌표를 직접 잡는다
     (`c.x = parent.x + dx`). 자식 순서는 `board.insertChild(index, node)`로 고친다.
   - 하단 고정용 Spacer는 `layoutGrow`가 풀리므로 **높이를 계산해 명시**한다.
4. 화면 하나가 끝날 때마다:
   - `growType==="auto-height"` 텍스트를 전부 `resize`로 재계산시킨다 (hHug 클리핑 방지).
   - `export_shape`로 PNG를 찍어 스펙과 눈으로 대조하고, 어긋나면 즉시 고친다.
     **안 보고 쌓으면 마지막에 전부 어긋나 있다.**
   - 로그에 화면 항목을 추가한다 (중간 실패에도 로그가 남게).
5. (재진입 모드) gap 목록이 왔으면 전체 재저작하지 않는다. **로그의 노드 ID로 해당
   부분만 타겟 수정**하고, 수정 항목을 로그의 `iteration`에 반영한다.
6. `04-screens-log.json`을 쓴다.

## 출력 형식

```json
{ "page": "", "iteration": 1,
  "screens": [
    { "screenId": "S01", "boardId": "", "boardName": "",
      "elements": { "요소 key(레이아웃 스펙과 동일)": "노드ID" },
      "satisfies": ["C01"] }
  ],
  "errors": [] }
```

## 금지

- Page 인자 없이 저작 시작 금지. 작업 Page 밖에 쓰기 금지.
- 입력에 없는 값을 지어내지 않는다. 레이아웃 스펙에 없는 화면·요소를 만들지 않는다.
- **특정 PRD 전용 하드코딩 금지.** 심사용 PRD는 미공개다.
- 다른 단계의 출력 파일을 쓰지 않는다. `03-library-log.json`을 수정하지 않는다.
