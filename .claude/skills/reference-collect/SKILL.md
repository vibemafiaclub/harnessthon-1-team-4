---
name: reference-collect
description: 도메인 키워드·퍼소나 요약·체크리스트를 받아 유사 서비스 UX 패턴을 수집하고 referencePatterns JSON 블록을 반환한다. stage-2-context에서 호출한다.
triggers:
  - "/reference-collect"
  - "레퍼런스 수집"
  - "유사 서비스 패턴"
---

# reference-collect 스킬

`00-clarify.md`에서 추출한 유저 문제를 기반으로, 동일한 문제를 푸는 외부 서비스를 찾아
UX 패턴을 수집한다. 결과는 `01-context.json`의 `referencePatterns` 필드에 들어간다.

## 입력 (인자로 받는다)

- `domain`: 도메인 설명 (예: "주식 입문자를 위한 모바일 투자 앱")
- `personas`: 퍼소나 목표·페인포인트 요약 (00-clarify.md에서 추출)
- `journey`: 핵심 유저 저니 단계 목록 (시작→중간→완료)
- `checklist`: 체크리스트 항목 목록 (어떤 화면/요소가 필요한지)

인자를 받지 못했으면 `00-clarify.md`를 직접 읽어 추출한다.

## 절차

### 1. 유저 문제 정의
입력(또는 00-clarify.md)에서 다음을 추출한다:
- 유저가 해결하려는 핵심 문제 (1문장)
- 도메인 성격 (카테고리, 대상 사용자층)
- 체크리스트에서 보이는 필수 화면·기능 유형

특정 PRD 고유명사(서비스명·지역명)에 의존하지 않는다. 도메인 성격으로 추론한다.

### 2. 유사 서비스 선정 (3~5개)
퍼소나·저니·도메인 키워드를 기반으로 동일 문제를 푸는 앱을 LLM 지식으로 도출한다.

선정 기준:
- 같은 유저 문제(퍼소나·저니)를 해결하는 서비스 우선
- 모바일 앱 우선 (스크린샷 수집 가능)
- 국내·해외 섞어 선정 (도메인 관습 다양성)

### 3. 앱별 접근 시도 (폴백 순서 엄수)

각 앱마다 아래 순서로 시도한다. 성공하면 다음으로 넘어가지 않는다.

**1차: Playwright → Google Play Store 웹**
```
URL: https://play.google.com/store/search?q={키워드}&c=apps
→ 앱 목록 확인, 상위 결과의 앱 상세 페이지 접근
URL: https://play.google.com/store/apps/details?id={앱id}
→ 스크린샷 이미지 URL 수집
→ 저장: docs/artifacts/refs/{앱명}/screenshot_{n}.png
```

**2차: Playwright → 공식 랜딩 페이지**
```
→ LLM 지식으로 공식 URL 추론
→ 페이지 스크린샷: docs/artifacts/refs/{앱명}/landing.png
→ 화면 목록·UI 요소 텍스트 추출
```

**3차: WebSearch**
```
쿼리 예시:
- "{앱명} UX pattern analysis"
- "{앱명} mobile app UI screenshot"
- "{도메인} mobile app UI best practices"
→ 텍스트 기반으로 패턴 추출 (PNG 없음)
→ accessMethod: "websearch"
```

**최종 폴백: LLM 훈련 데이터 기반**
```
→ LLM 지식으로 해당 앱의 알려진 UX 패턴 서술
→ accessMethod: "llm-knowledge"
→ screenshots: []
```

### 4. 패턴 추출 (앱마다)
접근 성공 시 스크린샷 시각 분석, 실패 시 텍스트 기반으로 추출:

| 추출 항목 | 설명 | stage-3-plan 활용 |
|---|---|---|
| `screens` | 주요 화면 목록 | layouts 설계 근거 |
| `layoutPatterns` | 레이아웃 패턴 서술 | children 배치 결정 |
| `components` | 업계 표준 컴포넌트 이름 | components.json 네이밍 |
| `colorConventions` | 색 관습 (상승/하락 등) | tokens.json color 설정 |
| `interactions` | 핵심 인터랙션 패턴 | 오버레이·인터랙션 설계 |

### 5. Synthesized 섹션 생성
앱별 결과를 종합해 공통 패턴을 뽑는다.
stage-3-plan은 개별 앱 결과보다 이 `synthesized`를 **우선 참조**한다.

## 출력 형식

```json
{
  "referencePatterns": {
    "domain": "stock trading for beginners",
    "userProblem": "투자 초보자가 종목을 발견하고 매수까지 가는 경험",
    "apps": [
      {
        "name": "Toss증권",
        "accessMethod": "playwright | websearch | llm-knowledge",
        "screenshots": ["docs/artifacts/refs/toss/screenshot_1.png"],
        "screens": ["관심종목 목록", "종목 상세", "주문 시트"],
        "layoutPatterns": [
          "차트 상단 고정",
          "하단 고정 주문 버튼 (CTA)",
          "탭으로 차트 기간 전환"
        ],
        "components": ["StockRow", "PriceChart", "OrderSheet", "TabBar"],
        "colorConventions": { "up": "적색 (#D14343)", "down": "청색 (#0071BC)" },
        "interactions": [
          "bottom sheet for order input",
          "swipe to dismiss sheet",
          "pinned bottom CTA"
        ]
      }
    ],
    "synthesized": {
      "commonScreens": ["watchlist", "stock-detail", "order-sheet", "search"],
      "commonComponents": ["StockRow", "PriceChart", "OrderSheet", "SearchBar"],
      "colorConvention": {
        "up": "적색 계열 (#D14343 또는 유사)",
        "down": "청색 계열 (#0071BC 또는 유사)"
      },
      "commonLayoutPatterns": [
        "종목 목록: 행마다 썸네일·이름·가격·등락",
        "종목 상세: 차트 상단, 주문 버튼 하단 고정",
        "주문 입력: bottom sheet"
      ],
      "commonInteractions": [
        "bottom sheet for input forms",
        "pinned bottom CTA button",
        "tab bar for range/category switch"
      ]
    }
  }
}
```

## 금지

- 특정 PRD 고유명사를 하드코딩하지 않는다. 도메인 성격으로 추론한다.
- Playwright 접근 실패 시 재시도 없이 즉시 다음 폴백으로 넘어간다.
- 수집 실패한 앱은 `accessMethod: "llm-knowledge"`로 기록하고 계속 진행한다 (중단하지 않는다).
- Penpot에 접근하지 않는다. 파일을 쓰지 않는다 (결과는 호출자인 stage-2가 병합해서 쓴다).
