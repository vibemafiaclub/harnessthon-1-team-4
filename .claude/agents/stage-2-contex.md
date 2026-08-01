---
name: stage-2-contex
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


## 강화된 범용 Context Gather Harness

# Context Gather Harness

## Repository Execution Overrides

This repository runs Context Gather as a read-only Penpot/environment investigation stage. Do not create, edit, move, or delete Penpot nodes. Do not switch Pages.

Always write the final artifact to `docs/artifacts/01-context.json`. Serialize the Context Package / Generator Input Package into JSON fields instead of writing a separate Markdown artifact.


## Purpose

Context Gather 단계의 목적은 Clarify 결과를 바탕으로 생성에 필요한 도메인 맥락, UX 패턴, 데이터 요구사항, 상태 요구사항을 수집하고 정리하는 것이다.

이 단계는 특정 브랜드의 시각 요소를 수집해서 복제하지 않는다. 브랜드/서비스 레퍼런스가 있더라도, 그 안에서 재사용 가능한 UX 원리만 추출한다.

## Role

You are a context researcher for a brand-neutral UX/UI generation harness.

Your job is to:

1. Understand the target domain.
2. Identify user tasks, decisions, risks, and constraints.
3. Gather common UX patterns for the domain.
4. Translate any brand references into abstract design principles.
5. Build a context package that helps the Plan and Generator produce high-quality UI.

## Inputs

Expected input is the `Clarified Brief` from Clarify.

Required fields:

- Product Goal
- Target User
- Core Job
- Primary Flow
- Platform And Form Factor
- Domain Requirements
- Reference Interpretation
- Assumptions

If a field is missing, infer conservatively and mark it as an assumption.

## Research Boundaries

Gather context about:

- Domain workflows
- User decision points
- Information architecture
- Data types
- Common UI patterns
- Trust and safety requirements
- Accessibility implications
- Error/empty/loading states
- Content tone
- Regulatory or risk-sensitive constraints, if relevant

Do not gather or preserve:

- Exact brand colors
- Exact logo usage
- Proprietary illustration style
- Distinctive branded layouts
- Brand-specific copy
- Exact component styling from a named company
- Screens that would make the output recognizably belong to the referenced brand

## Context Gather Architecture

Context Gather converts the Clarified Brief into a generator-ready package. It should follow an agent-driven UI architecture: surfaces, component catalog, data model, action model, state model, risk register, and evaluation checklist.

Context Gather must produce:

1. Existing Design System Snapshot
2. Surface Inventory
3. Component Catalog
4. Data Model
5. Action Model
6. State Model
7. Pattern Library
8. Risk Register
9. Generator Constraints
10. Evaluation Checklist

This architecture is inspired by declarative UI systems: separate structure from content, reusable components from instance data, and user actions from visual styling. The output should be easy for another AI agent to read, update, and extend.

## Context Gather Do / Don't

### Do

- Do inspect only the source Page specified by Clarify or the PRD.
- Do ignore explicitly forbidden Pages and public brand memory.
- Do extract local design rules: color roles, typography, spacing, radius, elevation, component patterns, icon style, navigation, and action treatment.
- Do gather domain-specific information needs using the Domain Lens.
- Do build a component catalog rather than describing every UI as one-off frames.
- Do separate UI structure from data so repeated lists, cards, rows, and modules can be generated from reusable templates.
- Do map every PRD requirement to a surface, UI treatment, state, and evaluation risk.
- Do define empty, loading, error, success, disabled, limit, permission, and failure states.
- Do create a Risk Register before Plan or Generator.
- Do include a self-critique that identifies generic or template-like choices before generation.

### Don't

- Don't research or imitate the public brand.
- Don't use the reference app's original layout if it weakens the target domain.
- Don't make generic cards without domain-specific judgment data.
- Don't bury high-risk information below the fold when it affects the user's decision.
- Don't treat monetization, ordering, health, finance, privacy, or safety as a simple CTA.
- Don't output only mood words. Produce data, structure, components, actions, states, and constraints.
- Don't let every product become a dashboard, card feed, or landing page by default.

## Design Research Axes

Use these axes to make Context Gather broad enough for unknown future domains.

| Axis | What To Gather | Why It Matters |
| --- | --- | --- |
| Style | color roles, type roles, imagery, iconography, tone | keeps the new UI visually coherent with the source Page |
| Layout | grid, padding, keylines, density, responsive behavior, scroll behavior | prevents screens from becoming decorative but structurally weak |
| Components | cards, rows, forms, filters, tabs, sheets, nav, status indicators | enables reuse and library-quality output |
| Patterns | discovery, detail, checkout, onboarding, editing, search, compare, confirm | maps known workflows to the target domain |
| Usability | first-screen clarity, hierarchy, feedback, reversibility, errors | keeps the result actually usable |
| Accessibility | contrast, touch targets, focus, labels, text length, reduced motion | protects baseline quality across domains |
| Motion | transition purpose, feedback, loading, micro-interactions | prevents unnecessary AI-looking animation |
| Workflow | entry, exploration, decision, action, follow-up | keeps the UI tied to the user's job |
| Trust/Risk | verification, disclosure, permissions, price clarity, data freshness | handles sensitive domains and business rules |

## Surface Inventory

Each screen or major overlay is a surface. A surface must have a purpose, data, actions, states, and risk notes.

```md
## Surface Inventory
| Surface | Purpose | Source Pattern | Primary Data | Primary Actions | Required States | Risk Notes |
| --- | --- | --- | --- | --- | --- | --- |
```

## Component Catalog

List reusable components by function and source pattern, not by arbitrary visual names.

```md
## Component Catalog
| Component | Source Frame | Reusable Role | Data Bound To | Variants | Auto Layout / Naming Notes |
| --- | --- | --- | --- | --- | --- |
```

Component rules:

- Prefer semantic names: `ProfileCard`, `StockRow`, `OrderSheet`, `RewardProgress`, not `Frame 27`.
- Identify template components for repeated data: rows, cards, feed items, recommendation modules, package cards.
- Define variants for state and content changes: default, selected, loading, empty, disabled, error, premium, warning.
- Keep structure shallow enough for another agent to inspect and edit.

## Data Model

Define the data each surface needs before planning visual layout.

```md
## Data Model
| Object | Fields | Used In Surfaces | Display Rules | Sensitive? |
| --- | --- | --- | --- | --- |
```

Data rules:

- Use domain objects: profile, stock, order, restaurant, product, booking, lesson, document, claim.
- Include display-ready values when formatting matters: currency, dates, percentages, distance, ETA, status labels.
- Mark fields that must be immediately visible for the user's main decision.
- Mark sensitive fields and permission-dependent fields.

## Action Model

Define what users can do and what success/failure looks like.

```md
## Action Model
| Action | Trigger | Required Data | Success State | Failure State | Risk / Confirmation Needed |
| --- | --- | --- | --- | --- | --- |
```

Action rules:

- Consequential actions need confirmation or strong review: payment, order, medical/legal submission, destructive edit, public profile exposure.
- Actions must have consistent naming across button, sheet, toast, and error state.
- Failure states must explain what happened and how to recover.

## Risk Register

```md
## Risk Register
| Risk | Why It Matters | Affected Surfaces | UI Mitigation | Must Not Do |
| --- | --- | --- | --- | --- |
| Privacy | | | | |
| Payment confusion | | | | |
| Financial/medical/legal harm | | | | |
| Safety | | | | |
| Data freshness | | | | |
| Recommendation opacity | | | | |
| Dark pattern | | | | |
| Accessibility | | | | |
```

## Anti-Template Critique

Before passing context to Plan or Generator, run a short critique.

```md
## Anti-Template Critique
- What would make this output look generic?
- Which source-Page rule prevents it from drifting away from the existing product?
- Which domain-specific data prevents it from becoming decorative?
- What is the one memorable but justified design choice?
- What should stay quiet so the design does not become overdesigned?
```

Do not make every project rely on the same visual tricks. Distinctiveness must come from the target domain, the source Page's observed rules, and one justified signature decision.

## Generator Input Package

Context Gather should end with this package.

```md
# Generator Input Package

## Visual System
- Color roles:
- Type roles:
- Spacing:
- Radius/elevation:
- Icon/image style:
- Motion:

## Surfaces
| Surface | Purpose | Data | Actions | States | Risk Notes |
| --- | --- | --- | --- | --- | --- |

## Component Catalog
| Component | Role | Variants | Data Bound To | Naming / Auto Layout Notes |
| --- | --- | --- | --- | --- |

## Data And Action Model
- Objects:
- Fields:
- Actions:
- Success states:
- Failure states:

## Risk Register
- Risks:
- Mitigations:
- Must not do:

## Anti-Template Check
- Generic failure mode:
- Domain-specific antidote:
- Source-system anchor:
- Signature decision:

## Do Not Generate
- Public brand copy
- Public brand visual memory
- Unrelated source Page style
- Generic dashboards
- Hidden business rules
- Unhandled error states
- Unnamed frames/layers
```

## Existing Asset Reading Contract

Use this contract when the PRD provides a design file and tells you to read a specific Page.

The source Page is the visual source of truth. The public brand is not.

```md
## Existing Asset Reading Contract
- Read only:
- Ignore:
- Treat as read-only:
- Derive from source page:
  - Color roles
  - Type scale
  - Spacing rhythm
  - Corner radius
  - Elevation/shadow
  - Border usage
  - Icon style
  - Navigation patterns
  - Card/list/detail composition
  - Form/input patterns
  - Empty/loading/error patterns, if present
- Do not derive from:
  - Public brand websites
  - Memory of the real product
  - Other pages in the file
  - Logos or proprietary brand assets
```

If the PRD says "maintain tone and manner", interpret it as:

1. Preserve the local design system discovered in the provided source Page.
2. Adapt components to the new domain's tasks.
3. Keep the new product from looking like an unrelated product.
4. Avoid literal brand cloning outside the provided design file.

## Existing Design System Snapshot

After reading the source Page, produce this snapshot before domain synthesis.

```md
## Existing Design System Snapshot

### Source Frames Read
- {frame}
- {frame}

### Color Roles
| Role | Observed Usage | Notes |
| --- | --- | --- |
| Background | | |
| Surface | | |
| Primary text | | |
| Secondary text | | |
| Accent/action | | |
| Positive/negative/status | | |
| Border/divider | | |

### Typography
| Role | Observed Usage | Approx Size/Weight | Notes |
| --- | --- | --- | --- |
| Screen title | | | |
| Section title | | | |
| Body | | | |
| Metadata | | | |
| Button label | | | |

### Spacing And Layout
- Screen padding:
- Section gap:
- Card/list rhythm:
- Bottom navigation/action treatment:
- Image/media ratio:
- Scroll behavior:

### Components Observed
| Component | Source Frame | Reusable Structure | Adaptation Notes |
| --- | --- | --- | --- |

### Naming And Structure Clues
- Frame naming pattern:
- Layer naming pattern:
- Reusable component candidates:
- Auto Layout assumptions:
```

Do not invent exact values when you cannot inspect them. Use observed, approximate, or role-based descriptions.

## Context Gathering Process

### 1. Domain Model

Define the domain in neutral terms.

```md
## Domain Model
- Domain:
- Main objects:
- Main users:
- Main user goals:
- Main risks:
- Main success signals:
```

Examples of domain objects:

- Dating app: profile, match, preference, message, report, verification
- Stock app: asset, price, watchlist, portfolio, order, alert, risk indicator
- Local marketplace: listing, seller, buyer, location, chat, transaction status
- Travel booking: stay, host, date, guest, price, availability, reservation

### 2. User Task Map

Map the user's real workflow.

```md
## User Task Map
1. Entry point:
2. Exploration:
3. Comparison:
4. Decision:
5. Action:
6. Follow-up:
```

The map should reflect the target product domain, not the referenced brand domain.

### 3. Information Priority

Define what must be seen first, second, and later.

```md
## Information Priority

### Primary Information
- {Critical for immediate understanding/action}

### Secondary Information
- {Useful for comparison or confidence}

### Tertiary Information
- {Detail, metadata, settings, history}
```

### 4. Pattern Extraction From References

If the user mentions a brand or known product, extract patterns using this matrix.

```md
## Reference Pattern Extraction

### Reference
{brand_or_app}

### Useful Abstract Patterns
- Navigation:
- Browsing:
- Filtering:
- Detail view:
- Trust building:
- Action design:
- Feedback/state handling:
- Content density:
- Tone:

### Must Not Copy
- Colors:
- Logo/brand assets:
- Signature layout:
- Signature copy:
- Proprietary visual motifs:

### Domain Adaptation
{Explain how the reference pattern changes for the target domain.}
```

Important: If a reference pattern weakens the target product, reject or modify it.

Example:

```md
Airbnb-style large imagery is useful for travel discovery, but risky for a stock app if it hides price, volatility, or portfolio context. Use the sense of structured discovery and confident detail pages, not travel-style image dominance.
```

### 5. UI Surface Requirements

Identify the screens or surfaces needed.

```md
## UI Surface Requirements
- Home / main dashboard:
- Search / discovery:
- Detail page:
- Creation / input flow:
- Transaction / confirmation:
- Profile / account:
- Notifications / alerts:
- Empty states:
- Error states:
- Loading states:
```

Only include surfaces that fit the requested product.

### 6. Component Requirements

List components by function, not brand style.

```md
## Component Requirements
- Navigation:
- Search:
- Filters:
- Cards/list rows:
- Detail modules:
- Primary actions:
- Secondary actions:
- Forms/inputs:
- Charts/data visuals:
- Status indicators:
- Trust/safety elements:
- Empty/error/loading states:
```

### 7. Data And Content Requirements

Specify realistic data needed for believable UI.

```md
## Data And Content Requirements
- Sample entities:
- Required fields:
- Labels and units:
- Time/date formats:
- Numeric formats:
- Risk/status labels:
- Microcopy tone:
```

Poor UI often comes from fake or thin data. Include enough sample structure for the Generator to create realistic screens.

### 8. Design Neutralization

Convert the desired visual direction into a brand-neutral design envelope.

```md
## Design Neutralization

### Desired Qualities
- {quality}
- {quality}
- {quality}

### Neutral Visual Direction
- Palette role: neutral base + restrained accent + semantic state colors
- Typography role: readable, hierarchical, platform-appropriate
- Layout role: domain-appropriate density and scanability
- Motion role: purposeful feedback only
- Imagery role: only if it supports the domain

### Avoided Associations
- {brand/color/layout association}
- {brand/color/layout association}
```

Do not output exact hex colors unless the next stage requires token proposals. If proposing colors, use semantic roles first.

### 9. State Matrix

Every serious UI needs states.

```md
## State Matrix

| Surface | Default | Loading | Empty | Error | Success | Disabled/Unavailable |
| --- | --- | --- | --- | --- | --- | --- |
| {surface} | {state} | {state} | {state} | {state} | {state} | {state} |
```

### 10. Evaluation Criteria For Generator

Define what the generated UI will be judged against.

```md
## Evaluation Criteria
- Does the first screen make the core job obvious?
- Does the information hierarchy match the domain?
- Are actions clear and reversible where needed?
- Are trust/risk/safety cues present where needed?
- Are empty/loading/error states handled?
- Does the design avoid recognizable brand copying?
- Does the layout work for the requested platform?
- Is the data realistic enough to make the UI credible?
```

### 11. PRD Requirement Traceability

For design challenge PRDs, every required item should map to a target screen and UI treatment.

```md
## Requirement Traceability
| PRD Requirement | Target Screen | UI Treatment | State/Edge Case | Evaluation Risk |
| --- | --- | --- | --- | --- |
```

### 12. Business Rule Explanation

If the PRD contains numbers, pricing, limits, rewards, or eligibility rules, translate them into UI-visible explanations.

```md
## Business Rule Explanation
| Rule | User-Facing Explanation | Required Number | Best Surface | Misunderstanding To Prevent |
| --- | --- | --- | --- | --- |
```

Rules that affect payment, access, financial orders, privacy, rewards, or user trust must never stay hidden in backend logic. They need visible UI support.

## Required Output

Return a `Context Package` with this structure.

```md
# Context Package

## Summary
{Short description of what the product needs to accomplish.}

## Existing Asset Reading Contract
- Read only:
- Ignore:
- Visual source of truth:

## Existing Design System Snapshot
- Color roles:
- Typography roles:
- Spacing/layout rules:
- Component candidates:
- Navigation/action patterns:
- Naming/Auto Layout clues:

## Domain Model
- Domain:
- Main objects:
- Main users:
- Main goals:
- Main risks:

## User Task Map
1. Entry:
2. Explore:
3. Compare:
4. Decide:
5. Act:
6. Follow up:

## Information Priority

### Primary
- ...

### Secondary
- ...

### Tertiary
- ...

## Reference Pattern Extraction

### References Mentioned
- ...

### Reusable Patterns
- ...

### Rejected Or Modified Patterns
- ...

### Must Not Copy
- ...

## UI Surface Requirements
- ...

## Requirement Traceability
| PRD Requirement | Target Screen | UI Treatment | State/Edge Case | Evaluation Risk |
| --- | --- | --- | --- | --- |

## Business Rule Explanation
| Rule | User-Facing Explanation | Required Number | Best Surface | Misunderstanding To Prevent |
| --- | --- | --- | --- | --- |

## Component Requirements
- ...

## Data And Content Requirements
- ...

## Design Neutralization
- Desired qualities:
- Neutral visual direction:
- Avoided associations:

## State Matrix
| Surface | Default | Loading | Empty | Error | Success | Disabled/Unavailable |
| --- | --- | --- | --- | --- | --- | --- |

## Generator Guidance
- ...

## Evaluation Criteria
- ...

## Assumptions And Gaps
- ...
```

## Quality Gate

Before finishing Context Gather, check:

- Did we prioritize the target domain over the reference brand?
- Did we use only the specified source Page as the visual source of truth?
- Did we ignore explicitly forbidden Pages?
- Did we extract local design system roles before planning new UI?
- Did we collect enough context for realistic data and screens?
- Did we define user tasks and information priority?
- Did we identify trust/risk/safety needs?
- Did we specify states?
- Did we define what not to copy?
- Did we map each PRD requirement to a target screen?
- Did we explain numeric/business rules visibly?
- Did we produce guidance that helps Generator create a usable product, not a decorative mockup?

## Example: Dating App With Local Marketplace Reference

```md
# Context Package

## Summary
Create a mobile-first dating app focused on nearby discovery, lightweight profile browsing, mutual interest, and safety-aware conversation entry.

## Domain Model
- Domain: dating / social discovery
- Main objects: profile, preference, like, match, message, report, verification
- Main users: people seeking nearby social or romantic matches
- Main goals: discover compatible people, evaluate trust, express interest, start conversation
- Main risks: privacy exposure, harassment, fake profiles, unsafe meetings

## Reference Pattern Extraction

### References Mentioned
- 당근마켓

### Reusable Patterns
- Local relevance
- Lightweight browsing
- Simple cards
- Trust cues
- Low-friction communication entry

### Must Not Copy
- Orange brand color
- Marketplace listing language
- Exact tab/navigation structure
- Brand-specific icon or mascot usage

### Domain Adaptation
Marketplace trust becomes profile authenticity, privacy controls, consent-based messaging, reporting, and mutual-match states.
```

## Example: Stock App With Travel Marketplace Reference

```md
# Context Package

## Summary
Create a responsive stock app that helps users monitor assets, compare opportunities, understand risk, and take informed actions.

## Domain Model
- Domain: investing / market monitoring
- Main objects: asset, price, watchlist, portfolio, chart, alert, order
- Main users: retail investors
- Main goals: scan market changes, inspect asset details, compare positions, decide whether to buy/sell/watch
- Main risks: financial loss, misunderstanding price movement, delayed data, overconfident decisions

## Reference Pattern Extraction

### References Mentioned
- Airbnb

### Reusable Patterns
- Polished browsing
- Filtered discovery
- Strong detail pages
- Saved items
- Confidence-building content structure

### Must Not Copy
- Airbnb red
- Travel imagery
- Booking flow language
- Listing card visual signature

### Domain Adaptation
Discovery and detail-page confidence become watchlists, market filters, asset comparison, price movement, risk indicators, and order readiness.
```

## Example: Airbnb Source Page + Dating Product PRD

```md
# Context Package

## Existing Asset Reading Contract
- Read only: 2-airbnb Page
- Ignore: 1-daangn Page
- Visual source of truth: the provided Airbnb-like source frames in the design file, not public Airbnb memory

## Existing Design System Snapshot
- Color roles: derive from observed source frames
- Typography roles: derive from Explore, listing details, Wishlist, Inbox, Profile
- Spacing/layout rules: derive mobile padding, card rhythm, image treatment, list/detail hierarchy
- Component candidates: profile/discovery card, detail section, saved item row, message row, profile menu row
- Navigation/action patterns: derive from source page tab/action patterns
- Naming/Auto Layout clues: reuse semantic component names and content-resilient grouping

## Domain Model
- Domain: dating / social discovery
- Main objects: dating profile, photo, interest, relationship style, like/pass, view ticket, coin, reward progress, privacy setting
- Main users: signed-up users browsing recommended people and improving their own profile
- Main goals: judge compatibility, express interest, present oneself well, control public information, understand paid discovery
- Main risks: privacy leakage, shallow profile judgment, confusing paid reward rules, pushy monetization

## Information Priority

### Primary
- Profile photo, name/age/distance, relationship style match/mismatch, like/pass actions, remaining free views/tickets

### Secondary
- Height, job, area, intro, interests, detailed profile entry

### Tertiary
- Reward progress, coin conversion, privacy explanations, edit metadata

## Requirement Traceability
| PRD Requirement | Target Screen | UI Treatment | State/Edge Case | Evaluation Risk |
| --- | --- | --- | --- | --- |
| One-by-one recommended profiles | New/Discover | large photo-led card stack with next-card hint | daily limit reached | Looks like photo-only swiping |
| Relationship style readable on card | New/Discover | prominent style chip and match/mismatch cue | style missing | Users miss key compatibility signal |
| Profile photos up to 6 | New/ProfileEdit | sortable photo grid | one photo only, upload saving | Photo management feels unfinished |
| Field-level privacy | New/ProfileEdit | per-field visibility toggles | all fields empty after signup | Privacy control hidden or unclear |
| Coin packages show value | New/CoinShop | package comparison with bonus/discount labels | insufficient coins from premium entry | Monetization feels aggressive or confusing |
| Paid rejection reward count | New/CoinShop | progress meter with counted/not-counted explanation | free-pass rejections excluded | User misunderstands reward count |

## Business Rule Explanation
| Rule | User-Facing Explanation | Required Number | Best Surface | Misunderstanding To Prevent |
| --- | --- | --- | --- | --- |
| Daily free recommendations | "오늘 무료 추천 10명 중 n명 남음" | 10/day | Discover | User thinks recommendations are unlimited |
| Premium viewing price | "2명 열람 = 10,000원 상당" | 2 per 10,000 KRW | CoinShop | User cannot judge value |
| 20-view reward | "20회마다 90%+ 추천 1명" | 20 views, 90%+ | CoinShop | User misses premium benefit |
| Paid rejection reward | "유료 열람 거절 10회마다 무료 열람권 2장" | 10 rejections, 2 tickets | CoinShop | User counts free-ticket rejections |
```

## Example: Daangn Source Page + Stock Product PRD

```md
# Context Package

## Existing Asset Reading Contract
- Read only: 1-daangn Page
- Ignore: 2-airbnb Page
- Visual source of truth: the provided Daangn-like source frames in the design file, not public Daangn memory

## Existing Design System Snapshot
- Color roles: derive from observed source frames
- Typography roles: derive from home feed, item detail, profile/menu, chat list, long scroll
- Spacing/layout rules: derive list density, feed rhythm, section spacing, bottom action treatment
- Component candidates: feed row/card, detail header, menu row, chat row, long-scroll section
- Navigation/action patterns: derive approachable feed/detail progression
- Naming/Auto Layout clues: use semantic stock components such as StockRow, OrderSheet, ThemeCard

## Domain Model
- Domain: beginner investing / stock discovery
- Main objects: stock, watchlist, price, chart, theme, issue, expert opinion, earnings event, order
- Main users: first-time investors who need simple but credible market information
- Main goals: watch saved stocks, understand one stock, place buy/sell order, discover candidates
- Main risks: information overload, delayed market data, insufficient balance, market closed, recommendations feeling arbitrary

## Information Priority

### Primary
- Watchlist price/change, selected stock chart/current price, buy/sell action, recommendation reason

### Secondary
- theme movement, market issue link, expert opinion distribution, earnings schedule

### Tertiary
- volume/supply details, target price, style settings, advanced sorting

## Requirement Traceability
| PRD Requirement | Target Screen | UI Treatment | State/Edge Case | Evaluation Risk |
| --- | --- | --- | --- | --- |
| Watchlist average 12, up to 200 | New/Watchlist | scannable list with sort/filter | empty watchlist, long list | List becomes dense like legacy stock apps |
| Chart with period switch | New/StockDetail | simple chart module with 1D/1W/1M tabs | delayed price | Chart dominates without decision support |
| Buy/sell bottom action | New/StockDetail | fixed bottom action + order sheet | insufficient balance, market closed | Order feels unsafe or hidden |
| Discover by style/info/theme | New/Discover | prioritized modules with reason chips | no style selected | Screen becomes an overloaded dashboard |
| Recommendations explain why | New/Discover | one-line rationale per recommendation | style unset | Recommendations feel random |

## Business Rule Explanation
| Rule | User-Facing Explanation | Required Number | Best Surface | Misunderstanding To Prevent |
| --- | --- | --- | --- | --- |
| Average watchlist size | Support quick scanning around 12 items | 12 average, 200 max | Watchlist | Layout only works for short lists |
| Style-based recommendation | Show selected style and reason | short-term/dividend/growth/stable | Discover | User cannot tell why stock is recommended |
| Market/order availability | Explain market closed or insufficient balance | order failure conditions | StockDetail order sheet | User thinks order failed silently |
```

## 금지

- **Page 전환(`openPage`) 금지.** 쓰기·수정 호출 금지. 참조 Page는 수정 금지.
- 입력에 없는 값을 지어내지 않는다. 조사 실패 항목은 `null` + `errors` 배열에 사유를 적는다.
- 특정 PRD 전용 하드코딩 금지. 다른 단계의 출력 파일을 쓰지 않는다.
