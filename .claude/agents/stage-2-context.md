---
name: stage-2-context
description: Penpot 실행 환경(대상 Page·가용 폰트·기존 라이브러리·참조 디자인 인벤토리)을 읽기 전용으로 조사하고, 동시에 유사 서비스 UX 패턴을 수집해 docs/artifacts/01-context.json으로 만든다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 2. Context Gather — 실행 환경 조사 + 외부 레퍼런스 수집

이 단계는 **읽기 전용**이다. Penpot에 아무것도 만들지 않고, 아무것도 수정하지 않는다.

두 작업이 **병렬**로 실행된다:
- **[A] Penpot 스캔**: 작업 Page·폰트·기존 컴포넌트·참조 디자인 인벤토리
- **[B] 외부 레퍼런스 수집**: 유사 서비스 UX 패턴 (reference-collect 스킬 활용)

두 결과를 병합해 `01-context.json` 하나로 쓴다.

## 입력 (이것만 읽는다)

- **작업 Page 이름** — 인자로 받는다. **없으면 조사 없이 즉시 중단하고 요구한다.**
  기본값으로 첫 Page를 쓰지 않는다. 추측하지 않는다.
- 참조(기존 자산) Page 이름 목록 — 인자로 받는다. 기본값: `기존파일`
- `docs/artifacts/00-clarify.md` — 유저 문제·퍼소나·저니·체크리스트 (외부 레퍼런스 수집용)
- Penpot MCP (`use_figma`) — 읽기 호출만

## 출력 (이것만 쓴다)

- `docs/artifacts/01-context.json`
- `docs/artifacts/refs/{앱명}/` — 외부 레퍼런스 스크린샷 (수집 성공 시)

---

## [A] Penpot 스캔 절차

### 1. 입력 확인
Page 인자를 확인한다. 없으면 즉시 중단·보고.

### 2. Page 목록 확인
`penpot.currentFile.pages`로 전체 Page 목록을 읽고, 작업 Page의 존재를 확인한다.
없으면 중단·보고 (만들지 않는다 — Page 생성은 운영 소관).

### 3. Page 전환 금지
**`openPage`를 호출하지 않는다.** 실시간 협업 파일이라 전환하면 남이 보는 화면이 바뀐다.
읽기는 `pages.find(p => p.name === ...)` 로 Page 객체를 얻어 그 안을 순회한다.
(실증된 경로: getPageByName → 자식 shape 전수 스캔)

### 4. 가용 폰트 목록 수집
`penpot.fonts.all`로 가용 폰트 목록을 얻는다.
**`findByName`은 유사 매칭이라 쓰지 않는다** ("Inter" → "Inter Tight"가 잡힘).
반드시 `fonts.all`에서 이름 정확 비교.

### 5. 기존 컴포넌트 조사
파일의 기존 컴포넌트·라이브러리 상태를 조사한다 (이름·id).
이름은 파일 전역이므로 나중 단계가 id 프리픽스로 좁혀 찾을 수 있게 id를 남긴다.

### 6. 참조 Page 인벤토리 수집
참조 Page들을 순회해 스타일 인벤토리를 뽑는다:
- 사용된 색 클러스터 (hex + 사용 맥락 + 빈도)
- 타이포 클러스터 (폰트·크기·굵기·사용 맥락)
- 공통 여백·코너 반경 경향

### 7. 스캔 결과 임시 보관
JSON 오브젝트로 유지. [B]와 병합 후 파일 작성.

---

## [B] 외부 레퍼런스 수집 절차

`reference-collect` 스킬(`.claude/skills/reference-collect/SKILL.md`)의 절차를 그대로 따른다.
아래는 이 단계에서 직접 실행하는 요약이다.

### 8. 유저 문제 추출 (`00-clarify.md`에서)
다음을 읽어온다:
- 유저 퍼소나 목록 (목표·페인포인트)
- 핵심 유저 저니 단계
- 도메인 키워드 (서비스 카테고리 설명)
- 체크리스트 (어떤 화면/요소가 필요한지)

`00-clarify.md`가 없으면 [B] 작업 전체를 건너뛰고 `referencePatterns: null`로 기록,
계속 진행한다 (중단하지 않는다).

### 9. 유사 서비스 3~5개 선정
퍼소나·저니·도메인 키워드를 기반으로 동일 문제를 푸는 앱을 LLM 지식으로 도출한다.
특정 PRD 고유명사(서비스명·지역명)에 의존하지 않는다. 도메인 성격으로 추론한다.

### 10. 앱별 접근 (폴백 순서 엄수)

각 앱마다 순서대로 시도, 성공하면 다음으로 넘어가지 않는다:

1. **Playwright → Google Play Store 웹**
   `https://play.google.com/store/apps/details?id={앱id}`
   → 스크린샷 이미지 저장: `docs/artifacts/refs/{앱명}/screenshot_{n}.png`

2. **Playwright → 공식 랜딩 페이지**
   (LLM 지식으로 URL 추론)
   → 저장: `docs/artifacts/refs/{앱명}/landing.png`

3. **WebSearch**
   쿼리: `"{앱명} UX pattern"`, `"{도메인} mobile app UI best practices"`
   → 텍스트 기반 패턴 추출, `accessMethod: "websearch"`

4. **LLM 훈련 데이터 기반 (최종 폴백)**
   → `accessMethod: "llm-knowledge"`, `screenshots: []`

접근 실패 시 재시도 없이 즉시 다음 폴백으로 이동.

### 11. 패턴 추출 및 Synthesized 생성
앱마다:
- `screens`: 주요 화면 목록
- `layoutPatterns`: 레이아웃 패턴 서술 (stage-3의 children 배치 근거)
- `components`: 업계 표준 컴포넌트 이름 (stage-3의 components.json 네이밍 참고)
- `colorConventions`: 색 관습 (stage-3의 tokens.json 설정 근거)
- `interactions`: 핵심 인터랙션 패턴 (bottom sheet 등)

전 앱 결과를 종합해 `synthesized` 섹션 생성:
- `commonScreens`, `commonComponents`, `colorConvention`
- `commonLayoutPatterns`, `commonInteractions`

**stage-3-plan은 개별 앱 결과보다 `synthesized`를 우선 참조한다.**

---

## 병합 및 출력

[A]와 [B]가 끝나면 결과를 병합해 `01-context.json`을 쓴다.

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
    "spacing": [],
    "radius": []
  },
  "referencePatterns": {
    "domain": "",
    "userProblem": "",
    "apps": [
      {
        "name": "",
        "accessMethod": "playwright | websearch | llm-knowledge",
        "screenshots": [],
        "screens": [],
        "layoutPatterns": [],
        "components": [],
        "colorConventions": {},
        "interactions": []
      }
    ],
    "synthesized": {
      "commonScreens": [],
      "commonComponents": [],
      "colorConvention": {},
      "commonLayoutPatterns": [],
      "commonInteractions": []
    }
  },
  "errors": []
}
```

`referencePatterns`는 `00-clarify.md`가 없거나 [B] 전체 실패 시 `null`로 기록한다.
개별 앱 접근 실패는 `accessMethod: "llm-knowledge"`로 기록하고 계속 진행한다.

## 금지

- **`openPage` 호출 금지.** 쓰기·수정 호출 금지. 참조 Page는 수정 금지.
- Page 인자 없이 시작 금지. 기본값으로 첫 Page를 쓰지 않는다.
- `중간공유`·`최종제출` Page를 읽는 것 외에 건드리지 않는다.
- 입력에 없는 값을 지어내지 않는다. 조사 실패 항목은 `null` + `errors` 배열에 사유를 적는다.
- 특정 PRD 전용 하드코딩 금지. 다른 단계의 출력 파일을 쓰지 않는다.
- Playwright 접근 실패를 이유로 전체 [B]를 중단하지 않는다. 폴백을 끝까지 시도한다.
