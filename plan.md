# 하네스 설계안 — 5-Phase 루프

> 작성: 2026-08-01 (채경완) · 상태: **팀 sync 대기**
> 목표: PRD를 입력받아 Penpot 디자인 파일을 무인 생성하는 하네스

## 핵심 아이디어

하네스 = **clarify → context gather → plan → generate → evaluate 인지 사이클을
반복 가능하게 만든 것**. 이 5개가 대분류(= sub agent)이고,
tokens / components / layouts / screens는 그 아래 소분류(= 산출물·스킬)다.

```
        ┌─────────────────────────────────────────────────┐
        │                                                 │
PRD ─► CLARIFY ─► CONTEXT GATHER ─► PLAN ─► GENERATE ─► EVALUATE ─► 합격 → 종료
        │              │             │         │            │
        │              │          tokens    tokens 등록      │
        │              │        components  components 저작  │
        │              │         layouts    screens 조립     │
        │              │                                    │
        └──────────── (gap 리포트를 들고 PLAN으로 재진입) ◄────┘
                        루프 상한: 3회
```

## 단계 정의

| 대분류 (agent) | 하는 일 | 소분류 | 산출물 (계약) |
|---|---|---|---|
| **1. Clarify** `stage-clarify` | PRD의 모호함 해소. 무인 실행이므로 질문 대신 **가정을 명시적으로 기록**. 화면 목록·필수 요소·충족 체크리스트 도출 | 요구사항, 가정 로그, 체크리스트 | `docs/artifacts/00-clarify.md` |
| **2. Context Gather** `stage-context` | 실행 환경 파악: 대상 Page 확정(인자), Penpot 가용 폰트, 기존 라이브러리 상태, (참고용) `1-daangn`·`2-airbnb` 스타일 인벤토리 | Penpot 상태, 참조 디자인 인벤토리 | `docs/artifacts/01-context.json` |
| **3. Plan** `stage-plan` | 저작 계획 수립. 소분류가 여기서 갈라짐: 토큰 체계 / 컴포넌트 목록·props / 화면별 레이아웃 스펙 | `tokens` `components` `layouts` | `docs/artifacts/02-plan/tokens.json`<br>`docs/artifacts/02-plan/components.json`<br>`docs/artifacts/02-plan/layouts/*.json` |
| **4. Generate** `stage-generate-library`<br>`stage-generate-screens` | Plan을 Penpot에 실체화. 소분류 순서 강제: **토큰 등록 → 컴포넌트 저작 → 화면 조립** (Penpot 쓰기는 직렬) | 토큰 등록, 컴포넌트 저작, 화면 조립 | Penpot 보드 + `docs/artifacts/03-build-log.json` (**생성 노드 ID 매핑**) |
| **5. Evaluate** `stage-evaluate` | `export_shape` 스크린샷 + 체크리스트 대조 + 채점 A 4축 루브릭 자체 채점 → **합격 or gap 리포트** | 시각 검증, 충족도 검증 | `docs/artifacts/04-eval-report.md` |

## 루프 메커니즘

루프의 주체는 `start/SKILL.md`(오케스트레이터). script처럼 동작하는 skill이다.

```
1. clarify → context           (서로 의존 없음 → 병렬 가능)
2. plan → generate → evaluate  (직렬)
3. eval-report에 gap이 있고 iteration < 3이면:
     gap 목록만 들고 plan 재진입 (전체 재계획 아님 — gap 항목만 수정 계획)
     → generate는 build-log의 노드 ID로 해당 부분만 타겟 수정
     → evaluate 재실행
4. 합격 or 상한 도달 → 종료, 최종 리포트
```

핵심 장치:

- **`03-build-log.json`의 노드 ID 매핑** — "화면 X의 컴포넌트 Y = Penpot 노드 zzz"를
  남겨야 루프 2회차에 전체 재저작 없이 **증분 수정**이 가능하다.
- **Clarify의 가정 로그** — 무인 실행이라 되물을 수 없으므로
  "PRD에 없어서 이렇게 가정했다"를 기록으로 대체. 심사 B "지침의 구체성" 어필 포인트.

## 설계 원칙

1. **각 단계 = sub agent 1개** (AGENTS.md 강제). 오케스트레이터는 직접 저작하지 않는다.
2. **단계 간 계약 = 번호 붙은 파일** (`docs/artifacts/`). 다음 단계는 상류 산출물과
   PRD만 읽는다.
3. **반하드코딩**: agent 프롬프트에 도메인 단어("당근마켓", "홈 화면" 등) 금지.
   오직 PRD·상류 산출물에서 도출한다. 심사용 PRD는 미공개다.
4. **Penpot 쓰기는 직렬**: MCP 연결이 하나뿐이고 실시간 협업 파일이므로
   동시 쓰기 금지. 병렬은 파일 산출물 단계(clarify/context/plan)에만 적용.
5. **모든 저작 단계는 Page 이름을 인자로 받는다** (penpot-design STEP 0 게이트).
   Page 미지정 시 즉시 중단하고 요구한다.
6. **QA 루프 상한 3회** — 무한 수정 루프 방지.

## 재현성 검증 방법

당근 화면을 역산한 예시 PRD + `2-airbnb` 역산 PRD, **서로 다른 도메인 2개**로
하네스를 각각 실행해 둘 다 통과하면 재현성 입증. 이 2회 실행 로그가
심사 어필 자료가 된다.

## 채점 축과의 대응

| 채점 항목 | 이 설계의 대응 장치 |
|---|---|
| A. 레이아웃·정렬 / 타이포·컬러 | tokens → components → screens 순서 강제로 일관성 보장 |
| A. PRD 충족도 | Clarify가 만든 체크리스트를 Evaluate가 대조 |
| A. 완성도·디테일 | Evaluate의 스크린샷 시각 검증 + 수정 루프 |
| B. 단계 분할의 타당성 | 인지 사이클 기반 5-Phase (대분류/소분류 위계) |
| B. 계약의 명료성 | 번호 붙은 artifact 파일 계약 |
| B. 재현성(반하드코딩) | 도메인 단어 금지 + 2개 도메인 PRD 검증 |
| B. 지침의 구체성 | 가정 로그, 루브릭 자체 채점 |
| B. 협업의 흔적 | agent 6개 = 1인 1개 담당 |

## 6명 배분 (agent 6개 = 1인 1개)

| 담당 | agent |
|---|---|
| 1명 | stage-clarify |
| 1명 | stage-context |
| 1명 | stage-plan |
| 2명 | stage-generate-library / stage-generate-screens |
| 1명 | stage-evaluate |

(+ 조장: `start/SKILL.md` 오케스트레이터 관리 — 공용 파일)

## Clarify 상세 설계 (2026-08-01 확정)

- **v1 범위 = 정규화 + 체크리스트 투영** (충돌 감지·질문 생성·게이트 판정은 v2)
- 정규화: `prds/prd-harness-test-corpus.md`의 공통 13항목 스키마. 창작 금지,
  근거 인용 필수, 없으면 `미기재`.
- **Evaluate용 체크리스트를 Clarify가 생성한다** — PRD 해석의 단일 출처 원칙.
  체크리스트는 원문이 아니라 정규화 결과에서만 투영 (presence / absence / global 3종).
- 구조: agent는 얇게(`.claude/agents/stage-clarify.md`), 방법론은 skill로
  (`.claude/skills/clarify/` — SKILL.md + schema.md). `/clarify <PRD경로>`로
  단독 실행 가능 → `prds/` 코퍼스 8종이 회귀 테스트 세트.

## 확정 시 해야 할 일

- [ ] 팀 sync에서 단계 구성·담당 확정 (새 단계 추가 = 조장 승인 사항)
- [ ] `start/SKILL.md` 단계 표·실행 순서 채우기 (조장)
- [ ] `.claude/agents/stage-*.md` 6개 뼈대 생성
- [ ] 예시 PRD 2개 작성 (당근 역산 / 에어비앤비 역산)
- [ ] 소분류 스킬 정비: 토큰 추출·정규화 방법은
  `docs/artifacts/design-tokens.daangn.json` 실증 결과를 스킬화

## 실증된 조각 (이미 검증됨)

- `1-daangn` Page 읽기 경로: `penpotUtils.getPageByName` → `shapeStructure` → 전수 스캔
- 토큰 추출·클러스터링: 848개 shape에서 색 17종 → 시맨틱 3계층, 타이포 17종 → 8단계
  스케일로 정규화 (`docs/artifacts/design-tokens.daangn.json`)
