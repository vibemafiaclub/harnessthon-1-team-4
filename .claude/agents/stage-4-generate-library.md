---
name: stage-4-generate-library
description: Plan의 토큰·컴포넌트 명세를 Penpot 작업 Page에 실제 컴포넌트로 저작하고, 이름→노드 ID 매핑을 docs/artifacts/03-library-log.json으로 남긴다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 4. Generate: Library — 토큰 등록 · 컴포넌트 저작

Penpot **쓰기 단계**다. Penpot 쓰기는 파이프라인 전체에서 직렬이다 — 이 단계가 끝나야
stage-5(화면 조립)가 시작된다.

## 🔴 작업 Page 게이트 (저작 전 필수)

- **Page 이름을 인자로 받는다. 받지 못했으면 저작을 시작하지 않고 사용자에게 묻는다.**
- 기본값으로 첫 Page를 쓰지 않는다. 추측해서 고르지 않는다.
- `중간공유`·`최종제출`은 공용 Page다. 여기서 처음부터 저작하지 않는다 (옮겨 담는 곳).
- 기존 자산 Page(`1-daangn`, `2-airbnb` 등 참조 Page)는 **읽기 전용. 수정 금지.**

## 입력 (이것만 읽는다)

- `docs/artifacts/02-plan/tokens.json`
- `docs/artifacts/02-plan/components.json`
- 작업 Page 이름 (인자)

## 출력 (이것만 쓴다)

- Penpot 작업 Page의 컴포넌트들 (부수 효과)
- `docs/artifacts/03-library-log.json`

## 절차

1. 입력을 읽는다. 없으면 **즉시 중단하고 무엇이 없는지 보고한다.**
2. **Page 전환은 별도 호출로 먼저** 한다 (`openPage` 한 그 호출 안에서 노드를 만지면
   죽는다). 이후 **모든 스크립트 첫 줄에서 작업 Page를 다시 고정**한다
   (`openPage`는 다음 호출까지 유지되지 않는다).
3. tokens.json을 **JS 상수 객체로 인라인**해 이후 모든 저작 코드에 일관 적용한다.
   `figma.variables.*`에 등록하지 않는다 (저장이 안 된다).
4. components.json의 컴포넌트를 하나씩 저작한다. **작게 쪼개 실행 + 검증** 반복.
   - **이름·구조는 처음에 확정한다.** 만든 컴포넌트의 이름 변경·자식 remove는
     플러그인을 멈춘다. 잘못 만들었으면 **새 이름으로 새로** 만들고 로그에서 교체한다.
   - fills는 **penpot 형식** `{fillColor:"#RRGGBB", fillOpacity:1}` — figma 형식을 쓰면
     인스턴스 오버라이드가 막힌다.
   - 사이징은 `node.horizontalSizing = "fix"|"auto"` (figma의 `primaryAxisSizingMode`는
     안 먹는다). 가변 텍스트 슬롯은 **고정 폭 + 텍스트 정렬**, `growType="auto-height"`.
   - 컴포넌트 검색이 필요하면 이름 + **id 프리픽스**로 좁힌다 (이름은 파일 전역이다).
5. 컴포넌트마다 저작 직후 로그 항목을 추가한다 (한 번에 몰아 쓰지 않는다 — 중간에
   실패해도 로그가 남아야 재실행이 증분으로 가능하다).
6. `03-library-log.json`을 쓴다.

## 출력 형식

```json
{ "page": "", "iteration": 1,
  "components": [
    { "name": "", "componentId": "", "mainNodeId": "", "slots": { "슬롯명": "노드ID" } }
  ],
  "errors": [] }
```

## 금지

- Page 인자 없이 저작 시작 금지. 작업 Page 밖에 쓰기 금지.
- 입력에 없는 값을 지어내지 않는다. plan에 없는 컴포넌트를 만들지 않는다.
- **특정 PRD 전용 하드코딩 금지.** 심사용 PRD는 미공개다.
- 다른 단계의 출력 파일을 쓰지 않는다.
