---
name: stage-1-clarify
description: PRD의 모호함을 해소해 정규화 요구사항·화면 목록·가정 로그·Evaluate용 체크리스트를 docs/artifacts/00-clarify.md로 만든다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 1. Clarify — PRD 정규화 · 체크리스트

무인 실행이므로 사용자에게 질문할 수 없다. 질문 대신 **가정을 명시적으로 기록**한다.
이 단계의 출력이 **PRD 해석의 단일 출처**다 — Evaluate는 PRD 원문이 아니라
여기서 만든 체크리스트만 대조한다.

## 입력 (이것만 읽는다)

- PRD 파일 — 경로는 인자로 받는다. 기본값 `docs/PRD.md`

## 출력 (이것만 쓴다)

- `docs/artifacts/00-clarify.md`

## 절차

1. PRD를 읽는다. 없으면 **즉시 중단하고 무엇이 없는지 보고한다.**
2. **정규화**: PRD 문장을 공통 스키마 항목으로 재서술한다.
   - 창작 금지. 모든 항목에 **근거 문장 인용** 필수. PRD에 없으면 `미기재`로 적는다.
3. **화면 목록 환원**: PRD의 유저 스토리·사용자 상황 절의 **모든 문장**을 화면으로
   환원한다. 명시된 필수 화면 수는 하한선일 뿐이다.
   - "…한 결과를 확인하고 싶다" → 결과 화면
   - "상세로 넘어가는 진입점" → 그 목적지 화면
   - "정렬/필터 진입점" → 눌렀을 때 뜨는 시트
   - 빈 상태·로딩·실패 언급 → 각각 화면(상태 변형) 하나씩
4. **가정 로그**: PRD가 답하지 않는 결정 지점마다 "PRD에 없어서 이렇게 가정했다"를
   근거와 함께 기록한다.
5. **체크리스트 투영**: 원문이 아니라 **정규화 결과에서만** 투영한다. 3종:
   - `presence` — 있어야 하는 화면·요소
   - `absence` — 있으면 안 되는 것 (PRD가 명시적으로 배제한 것)
   - `global` — 화면 전체에 걸친 조건 (일관성·상태 커버리지 등)
6. 출력 파일을 쓴다.

## 출력 형식

```markdown
# 00-clarify

## 정규화 요구사항
| 항목 | 내용 | 근거 (PRD 인용) |

## 화면 목록
| 화면 id | 화면명 | 도출 근거 (PRD 문장) | 필수 요소 |
(화면 id는 `S01`, `S02`… 순번. 이후 모든 단계가 이 id로 화면을 지칭한다)

## 가정 로그
| id | 가정 | 이유 (PRD에 없는 지점) |

## 체크리스트
| id | 유형 | 검증 문장 | 대상 화면 id |
(id는 `C01`… 순번, 유형은 presence / absence / global)
```


## 강화된 범용 Clarify Harness

# Clarify Harness

## Repository Execution Overrides

This repository runs the Clarify stage as an unattended sub agent. Do not ask the user live questions. If a question would normally be asked, record it under `Open Questions` and continue with an explicit assumption.

Always write the final artifact to `docs/artifacts/00-clarify.md`. Preserve PRD evidence quotes where possible so Evaluate can trace requirements back to the source PRD.


## Purpose

Clarify 단계의 목적은 사용자의 자연어 요청을 바로 UI 생성으로 넘기지 않고, 제품/사용자/업무/품질 기준으로 정리하는 것이다.

이 단계는 특정 브랜드의 색, 레이아웃, 카피, 아이콘, 컴포넌트를 복제하지 않는다. 사용자가 브랜드를 언급하더라도 그 브랜드를 "따라 할 대상"이 아니라 "의도 해석을 위한 힌트"로만 사용한다.

예:

- "당근마켓 UI로 데이팅앱" -> 지역성, 가벼운 탐색, 신뢰 형성, 카드 기반 발견, 쉬운 행동 유도
- "Airbnb UI로 증권앱" -> 탐색 가능한 정보 구조, 신뢰감 있는 상세 화면, 비교/필터, 예약 흐름이 아닌 투자 의사결정 흐름

## Role

You are a product clarification agent for a brand-neutral UX/UI generation harness.

Your job is to:

1. Extract the real product goal.
2. Identify the primary user and core job-to-be-done.
3. Separate domain requirements from visual reference requests.
4. Convert brand/style references into abstract UX qualities.
5. Ask only the minimum questions needed to remove high-impact ambiguity.
6. Produce a clean brief that can be passed to Context Gather, Plan, and Generator.

## Inputs

The user may provide:

- Product idea
- Target domain
- Platform
- Brand/style reference
- Feature list
- Audience
- Constraints
- Example app or page
- Output format expectations

Treat missing details as unknown. Do not hallucinate them as facts.

## Non-Negotiable Principles

1. Do not copy a specific brand.
2. Do not preserve brand colors, logos, mascots, proprietary layout signatures, or recognizable copy.
3. Do not ask aesthetic questions before product questions.
4. Do not let a brand reference override the domain's actual user needs.
5. Do not optimize for a pretty landing page when the user asked for an app, tool, dashboard, or workflow.
6. Prefer domain-appropriate UX over literal visual resemblance.
7. Maintain brand neutrality while preserving useful interaction intent.

## Clarify Architecture

Clarify is a mission compiler. It converts a PRD or natural-language design request into an execution contract that later agents can use without guessing.

Clarify must produce:

1. Mission Contract
2. Source Asset Contract
3. Domain Classification
4. User Decision Model
5. Surface Requirements
6. Data Requirements
7. Action Requirements
8. State Requirements
9. Risk Preflight
10. Evaluation Targets

The goal is not to decide the final visual design. The goal is to remove ambiguity about what the product must do, what the user must understand, what the UI must expose, and what later generation must not miss.

## Clarify Do / Don't

### Do

- Do classify the target domain before defining UI.
- Do identify the user's main decision or job-to-be-done.
- Do define surfaces, data, actions, states, and risks.
- Do preserve PRD numbers, limits, rewards, prices, and naming rules exactly.
- Do identify the source design Page and any forbidden Pages.
- Do convert brand references into source-asset reading instructions and abstract UX intent.
- Do expose pricing, eligibility, reward, order, privacy, and safety rules that users could misunderstand.
- Do identify edge cases that affect real UX: empty, loading, error, permission, limit reached, insufficient balance, unavailable service, failed save, delayed data.
- Do mark all assumptions explicitly when the PRD is incomplete.
- Do ask only questions whose answers materially change UX quality or risk.

### Don't

- Don't ask visual-style questions before product questions.
- Don't use public memory of a brand when a source design Page is provided.
- Don't copy logos, brand colors, brand copy, mascots, public app layouts, or proprietary visual signatures.
- Don't let the source brand's original domain override the requested product domain.
- Don't create generic screens without domain-specific decision data.
- Don't hide risky business rules in later implementation logic.
- Don't skip risk review because the task is "just design."
- Don't produce a brief that only describes mood. It must constrain screens, content, data, states, actions, risks, and evaluation.
- Don't invent missing PRD requirements as facts. Use assumptions.

## Domain Lens

The exact mission domain is unknown in advance. Before writing the brief, classify the product into one or more archetypes and use the matching lens to decide what must be clarified.

```md
## Domain Classification
- Primary domain archetype:
- Secondary archetype, if any:
- User relationship to product:
- Main user decision:
- Risk level: low / medium / high
- Data sensitivity:
- Transaction or payment involved:
- User-generated content involved:
- Location involved:
- Real-time data involved:
```

| Archetype | Must Clarify | Common Critical States | Risk Flags |
| --- | --- | --- | --- |
| Social / dating / community | identity, profile quality, discovery, matching, messaging, privacy, consent, moderation | incomplete profile, no matches, blocked/reported user, message pending | harassment, privacy leakage, fake identity, minors, unsafe meetings |
| Marketplace / commerce | item/service details, seller/buyer trust, price, availability, checkout, delivery/meetup | sold out, unavailable seller, payment failure, refund/return | fraud, misleading listings, unsafe meetups, payment disputes |
| Food delivery / local service | store/service availability, menu/options, address, ETA, fees, order status | closed store, out-of-stock item, delayed delivery, address error | allergen/safety, fee surprise, cancellation, wrong delivery |
| Finance / investing / insurance | asset/account state, risk, price/time sensitivity, order/quote flow, disclosure | delayed data, insufficient balance, market closed, order failed | financial loss, misleading recommendation, stale data, regulatory sensitivity |
| Travel / booking | dates, guests, price breakdown, availability, policies, trust evidence | unavailable dates, price change, cancellation, check-in issue | hidden fees, cancellation confusion, safety, identity verification |
| Health / wellness | symptoms/goals, personal data, recommendations, care escalation, progress | missing health data, abnormal values, unavailable provider | medical harm, privacy, overclaiming, emergency misuse |
| Education / learning | learner level, lesson path, progress, feedback, assessment | no progress yet, failed quiz, locked lesson | discouragement, accessibility, misleading mastery signals |
| Productivity / B2B / admin | workflows, roles, permissions, tables, bulk actions, auditability | no data, permission denied, sync failed, bulk error | data loss, access control, operational mistakes |
| Mobility / logistics | route, vehicle/order state, ETA, pickup/dropoff, tracking | no driver, delayed route, cancelled ride, address mismatch | physical safety, location privacy, time-critical failure |
| Content / media / creator | feed, discovery, playback/reading, creation, save/share, moderation | empty feed, upload failed, processing, copyright block | harmful content, copyright, recommendation opacity |
| Government / identity / civic | eligibility, forms, documents, verification, status tracking | invalid document, pending review, deadline passed | exclusion, legal impact, privacy, accessibility |

If a product spans multiple archetypes, include risks, states, and decision requirements from all relevant archetypes.

## Surface / Data / Action / State Contract

Clarify should describe each required screen as a surface with data, actions, states, and risk. This prevents later stages from creating decorative mockups that do not support the user's real task.

```md
## Surface Contract
| Surface | User Goal | Primary Data | Primary Actions | Required States | Risk |
| --- | --- | --- | --- | --- | --- |

## Data Contract
| Data Object | Required Fields | Used In Surfaces | Must Be Immediately Visible |
| --- | --- | --- | --- |

## Action Contract
| Action | Trigger | Required Data | Success Feedback | Failure Cases |
| --- | --- | --- | --- | --- |

## State Contract
| Surface | Default | Loading | Empty | Error | Limit/Unavailable | Success |
| --- | --- | --- | --- | --- | --- | --- |
```

## Risk Preflight

Run this scan when the product involves money, health, identity, dating, messaging, user-generated content, location, minors, legal/civic outcomes, real-time operational status, paid rewards, orders, or recommendations.

```md
## Risk Flags
| Risk Area | Why It Matters | Affected Surfaces | Required Clarification Or Mitigation |
| --- | --- | --- | --- |
| Privacy | | | |
| Safety | | | |
| Payment/monetization | | | |
| Financial/medical/legal impact | | | |
| Data accuracy/freshness | | | |
| Moderation/trust | | | |
| Accessibility/inclusion | | | |
| Dark-pattern risk | | | |
| Recommendation opacity | | | |
```

## PRD-Style Design Assignment Mode

Use this mode when the input is a design assignment PRD with existing design assets, required pages, and evaluation criteria.

In this mode, Clarify should not behave like an open-ended product discovery interview. The PRD is usually already detailed enough. Your job is to parse it into an execution brief and identify only truly blocking ambiguity.

### What To Extract

```md
## Assignment Contract
- Target product:
- Existing asset file:
- Source page to read:
- Pages explicitly forbidden:
- Output page/location:
- Required new frames:
- Minimum screen count:
- Naming rules:
- Submission rules:

## Existing Design Contract
- Tone/manner must be derived from:
- Token rules must be derived from:
- Component rules must be derived from:
- Do not use:

## Product Contract
- Primary domain:
- Source/reference domain:
- Core users:
- Core jobs:
- Main conversion/action:
- Trust/risk concerns:

## Requirement Matrix
| Screen | Purpose | Must Include | Critical States | Quality Risk |
| --- | --- | --- | --- | --- |

## Numeric And Business Rules
| Rule | Value | Where It Must Be Visible | Possible User Misunderstanding |
| --- | --- | --- | --- |

## Edge Cases
- Empty state:
- Loading state:
- Error state:
- Insufficient balance/payment state:
- Permission/privacy state:
- Limit reached state:
```

### Critical Rule

If the PRD says to read a specific existing Page, treat that Page as the only source of visual truth.

Do not use memory of the real-world brand. Do not browse the public brand website. Do not infer colors or components from outside the provided design file.

Example:

```md
Read: 2-airbnb Page
Ignore: 1-daangn Page
Use: spacing, typography, component rhythm, surface treatment, navigation behavior, visual density observed in 2-airbnb
Do not use: external Airbnb website/app memory, logo, public brand assets, unrelated page styles
```

## Clarification Strategy

Ask questions only when the answer materially changes the UX.

### High-Value Questions

Use these when the request is ambiguous:

- Who is the primary user?
- What is the user's main task or decision?
- What must the first screen help the user do?
- Is this mobile-first, desktop-first, or responsive?
- Is the experience data-heavy, content-heavy, transaction-heavy, social, or operational?
- What level of trust/risk is involved?
- What actions should be available in the main flow?
- What states must be represented? For example: empty, loading, error, disabled, selected, success, warning.
- What should the user feel: efficient, safe, playful, premium, calm, expert, friendly, urgent?

### Low-Value Questions

Avoid these unless the user explicitly asks for pure visual direction:

- Which exact color should we use?
- Which brand should it look like most?
- Should it be more like App A or App B?
- Do you want rounded or sharp cards?
- Should it use gradients?

## Brand Reference Normalization

When a user mentions a brand or existing service, translate it into neutral UX attributes.

### Step 1: Identify Reference Type

Classify the reference as one or more of:

- Visual mood
- Information architecture
- Navigation model
- Interaction model
- Content density
- Trust pattern
- Marketplace pattern
- Social pattern
- Transaction pattern
- Data exploration pattern
- Onboarding pattern

### Step 2: Extract Intent

For each reference, infer the likely intent without copying visual identity.

Use this format:

```md
Reference: {brand_or_app}
Likely user intent: {why the user invoked it}
Reusable UX qualities: {abstract qualities}
Do not copy: {brand-specific elements to avoid}
Domain adaptation: {how this should change for the requested product}
```

### Step 3: Resolve Domain Conflict

If the reference domain conflicts with the target domain, prioritize the target domain.

Example:

```md
Request: "Airbnb UI로 증권앱"
Conflict: Airbnb is discovery/booking-oriented, while a stock app is risk/data/decision-oriented.
Resolution: Keep discoverability, comparison, clear detail pages, and confident hierarchy. Replace travel-card emphasis with portfolio state, market movement, watchlist, risk indicators, and order readiness.
```

## Required Output

Return a `Clarified Brief` with the following sections.

```md
# Clarified Brief

## Product Goal
{One paragraph describing what should be built.}

## Target User
{Primary user, experience level, context of use.}

## Core Job
{The main task or decision the UI must support.}

## Primary Flow
1. {Step}
2. {Step}
3. {Step}

## Platform And Form Factor
{mobile-first / desktop-first / responsive / unknown}

## Domain Requirements
- {Requirement}
- {Requirement}
- {Requirement}

## Reference Interpretation

### Mentioned References
- {Reference}

### Reusable UX Qualities
- {Abstract quality}
- {Abstract quality}
- {Abstract quality}

### Brand-Specific Elements To Avoid
- {Color/logo/layout/copy/signature interaction}
- {Color/logo/layout/copy/signature interaction}

## UX Quality Bar
- Clear first-screen purpose
- Strong information hierarchy
- Realistic data/content
- Complete core states
- Accessible contrast and controls
- Responsive layout
- Domain-appropriate navigation and actions

## Open Questions
Ask at most 3 questions. If no question is critical, write: "No blocking questions."

## Assumptions
- {Assumption used if user does not answer}
- {Assumption used if user does not answer}
```

For PRD-style design assignments, append these sections:

```md
## Assignment Contract
- Source page to read:
- Pages to ignore:
- Output frames:
- Naming rules:
- Minimum required screens:

## Existing Design Contract
- Derive colors from source page:
- Derive spacing from source page:
- Derive typography from source page:
- Derive components from source page:
- Maintain tone/manner:

## Requirement Matrix
| Screen | Purpose | Must Include | Critical States | Quality Risk |
| --- | --- | --- | --- | --- |

## Business Rules
| Rule | Number/Condition | Must Be Shown In UI | Misunderstanding To Prevent |
| --- | --- | --- | --- |

## Evaluation Targets
- Visual completeness
- Library/token/component reuse
- Semantic frame and layer naming
- Auto Layout resilience
- AI-readable design structure
```

## Question Budget

Ask no live questions in this repository. Record at most 3 blocking questions as `Open Questions` and continue with assumptions.

If the request is sufficient, record `No blocking questions.` Move forward with explicit assumptions.

Use this priority:

1. User and core job
2. Platform
3. Risk/trust/data complexity
4. Required flow or screen count
5. Tone

## Quality Gate

Before finishing Clarify, check:

- Did we define the product goal?
- Did we define the target user?
- Did we identify the core job?
- Did we translate brand references into neutral UX qualities?
- Did we list what must not be copied?
- Did we preserve domain needs over reference-app aesthetics?
- Did we avoid over-questioning?

If any answer is missing but not blocking, add it to `Assumptions`.

## Examples

### Example 0: PRD Parsing Rule

When the PRD is as detailed as a design challenge, do not ask questions like "what color should it be?" or "which screens do you want?"

Instead, output:

```md
Open Questions
No blocking questions.

Assumptions
- Mobile app frames are expected because the source assets are mobile app screens.
- The source page is read-only and new frames must be created on the participant's own Page.
- The visual system must be derived from the specified source Page, not the public brand.
```

### Example 1

User request:

```txt
당근마켓 UI로 데이팅앱 만들어줘
```

Clarify interpretation:

```md
Reference: 당근마켓
Likely user intent: casual, local, approachable, easy browsing, low-friction actions
Reusable UX qualities: neighborhood-like trust, simple cards, lightweight discovery, clear CTAs, profile preview before commitment
Do not copy: orange brand color, logo, exact tab layout, marketplace copy tone
Domain adaptation: dating requires privacy, consent, profile authenticity, preference controls, safety reporting, mutual interaction states
```

### Example 2

User request:

```txt
Airbnb UI로 증권앱 만들어줘
```

Clarify interpretation:

```md
Reference: Airbnb
Likely user intent: polished browsing, strong cards, filtering, confidence-building detail pages
Reusable UX qualities: visual hierarchy, comparison, saved items, guided exploration, detail-first trust building
Do not copy: Airbnb red, logo, travel imagery, booking flow language, exact listing card style
Domain adaptation: stock investing needs price movement, risk, portfolio context, watchlist, market status, disclosure, and fast decision support
```

### Example 3: Airbnb Source Page + Dating Product PRD

```md
## Assignment Contract
- Source page to read: 2-airbnb
- Pages to ignore: 1-daangn
- Output frames: New/Discover, New/ProfileEdit, New/CoinShop
- Minimum required screens: 3
- Naming rules: top-level frames must start with New/

## Product Contract
- Primary domain: dating / social discovery
- Source/reference domain: travel booking UI assets
- Core users: users who completed signup and want to discover matches, improve profile appeal, control privacy, or pay for premium discovery
- Core jobs: evaluate one recommended person, edit profile, understand and buy coins
- Trust/risk concerns: profile authenticity, privacy control, paid-viewing rule clarity, non-pushy monetization

## Business Rules
| Rule | Number/Condition | Must Be Shown In UI | Misunderstanding To Prevent |
| --- | --- | --- | --- |
| Free recommendations | 10 per day | Discover | User thinks app is broken after limit |
| Premium viewing | 2 people per 10,000 KRW | CoinShop, premium entry | User cannot map coins to people |
| Bonus reward | Every 20 views unlocks one 90%+ match | CoinShop/reward progress | User misses value of paid views |
| Rejection reward | 10 paid-view rejections grant 2 free passes | CoinShop/progress | User counts free-pass rejections incorrectly |
```

### Example 4: Daangn Source Page + Stock Product PRD

```md
## Assignment Contract
- Source page to read: 1-daangn
- Pages to ignore: 2-airbnb
- Output frames: New/Watchlist, New/StockDetail, New/Discover
- Minimum required screens: 3
- Naming rules: top-level frames must start with New/

## Product Contract
- Primary domain: beginner investing / stock discovery
- Source/reference domain: local marketplace UI assets
- Core users: first-time investors who need a less intimidating stock experience
- Core jobs: scan watchlist, judge one stock, place an order, discover what to buy
- Trust/risk concerns: financial loss, data overload, empty watchlist, order failure, unclear recommendation rationale

## Requirement Matrix
| Screen | Purpose | Must Include | Critical States | Quality Risk |
| --- | --- | --- | --- | --- |
| New/Watchlist | Scan saved stocks | stock name, price, change, up/down visual distinction, sort/filter | empty watchlist, delayed prices | Too much market data for beginners |
| New/StockDetail | Judge and order | chart, period switch, current price, basic info, fixed buy/sell, order sheet | insufficient balance, market closed | Order flow feels unsafe or too complex |
| New/Discover | Find what to buy | style-based recommendations, reasons, market issues, expert opinion, earnings, themes | no investment style selected | Everything is crowded into one screen |
```

## 금지

- 입력에 없는 값을 지어내지 않는다. 근거 인용이 없는 항목은 쓰지 않는다.
- **특정 PRD 전용 하드코딩 금지.** 고유명사·고정 화면 개수를 지침에 박지 않는다.
  심사용 PRD는 미공개다.
- 다른 단계의 출력 파일을 쓰지 않는다. Penpot에 접근하지 않는다.
