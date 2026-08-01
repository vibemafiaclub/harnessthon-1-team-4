---
name: start
description: PRD를 입력받아 단계별 sub agent를 순서대로 호출해 Penpot 디자인을 완성하는 하네스 진입점. "/start", "시작해줘", "디자인 만들어줘", "PRD 실행", "하네스 돌려줘" 등에 트리거된다.
---

# start — 하네스 진입점 (5-Phase 루프)

> 🔒 **공용 파일입니다. 수정하려면 조장 승인이 필요합니다.**
>
> 설계 근거: `plan.md` (clarify → context → plan → generate → evaluate 인지 사이클)

## 입력

- PRD 경로 — 인자로 받는다. 기본값 `docs/PRD.md`
- **작업 Page 이름 — 매 실행마다 확인한다.** 없으면 묻고, 답을 받기 전엔 시작하지
  않는다. 기본값으로 첫 Page를 쓰지 않는다. `중간공유`·`최종제출`·기존 자산
  Page(`1-daangn`, `2-airbnb`)는 작업 Page로 받지 않는다 — 지정되면 되묻는다.

## 실행 원칙

1. **각 단계는 반드시 sub agent에게 위임한다.** 오케스트레이터가 직접 저작하지 않는다.
2. 각 sub agent에게 **입력(읽을 파일)·출력(쓸 파일)·작업 Page 이름**을 명시적으로 넘긴다.
3. **의존관계가 없는 단계는 병렬로** 호출한다. 단, **Penpot 쓰기는 직렬**이다
   (실시간 협업 파일 + MCP 연결 1개) — 병렬은 파일 산출물 단계에만 적용한다.
4. 중간 산출물은 전부 `docs/artifacts/`에 남긴다. 남지 않으면 다음 단계가 읽을 게 없다.
5. 한 단계가 출력 파일을 남기지 못했으면 **다음 단계로 넘어가지 않는다.** 멈추고 보고한다.

## 단계 정의

| # | 단계 | sub agent | 입력 | 출력 | 담당자 | 병렬 가능 |
|---|---|---|---|---|---|---|
| 1 | Clarify | `stage-1-clarify` | PRD | `docs/artifacts/00-clarify.md` | 미정 | ✅ (2와) |
| 2 | Context Gather | `stage-2-contex` | Page 이름, Penpot(읽기) | `docs/artifacts/01-context.json` | 미정 | ✅ (1과) |
| 3 | Plan | `stage-3-plan` | 00, 01 (+재진입 시 gap) | `docs/artifacts/02-plan/` (tokens·components·layouts) | 미정 | — |
| 4 | Generate: Library | `stage-4-generate-library` | 02-plan, Page 이름 | Penpot 컴포넌트 + `docs/artifacts/03-library-log.json` | 미정 | — (Penpot 쓰기 직렬) |
| 5 | Generate: Screens | `stage-5-generate-screens` | 02-plan/layouts, 03, Page 이름 (+gap) | Penpot board + `docs/artifacts/04-screens-log.json` | 미정 | — (Penpot 쓰기 직렬) |
| 6 | Evaluate | `stage-6-evaluate` | 00, 04, Page 이름(읽기) | `docs/artifacts/05-eval-report.md` | 미정 | — |
| V | Verify | `stage-verify-penpot` | Page 이름, 00, artifacts | `docs/artifacts/99-verify.md` | 조장 | — |

## 실행 순서

```
0. 게이트: 작업 Page 이름 확정 (없으면 묻고 대기. 시작하지 않는다)
1. stage-1-clarify ∥ stage-2-contex   (병렬 — 서로 의존 없음)
   → 00-clarify.md, 01-context.json 생성 확인
2. stage-3-plan → 02-plan/ 생성 확인 (layouts가 화면 목록 수만큼 있는지)
3. stage-4-generate-library → 03-library-log.json 확인   (직렬)
4. stage-5-generate-screens → 04-screens-log.json 확인   (직렬)
5. stage-6-evaluate (iteration 번호를 넘긴다) → 05-eval-report.md 확인
6. 루프 판정:
   - 판정이 합격 → 7로
   - 불합격 && iteration < 3 →
       gap 표만 들고 stage-3-plan 재진입 (전체 재계획 아님)
       → 수정된 계획 범위에 따라 stage-4 / stage-5를 gap의 노드 ID 타겟으로 재호출
       → stage-6-evaluate 재실행 (iteration + 1) → 다시 6
   - 불합격 && iteration == 3 → 루프 중단, 남은 gap을 최종 리포트에 포함하고 7로
7. stage-verify-penpot 호출 (아래 참조)
```

## 마지막 단계 — 검증 (고정, 삭제 금지)

모든 단계가 끝나면 **항상** `stage-verify-penpot` 을 호출한다.

- 지정 Page에 board/frame이 1개 이상 있는가
- PRD가 요구한 화면이 전부 있는가
- 각 단계 산출물이 `docs/artifacts/`에 남아 있는가

결과는 `docs/artifacts/99-verify.md`. **실패 항목이 있으면 완료를 선언하지 않는다.**
실패 항목의 담당 단계를 다시 호출하고, 재실행 후에도 실패하면 무엇이 왜 비었는지
사용자에게 보고한다.

## 완료 조건

- Penpot 파일의 **지정된 Page**에 화면이 실제로 만들어져 있다
- 각 단계의 중간 산출물이 `docs/artifacts/`에 남아 있다
- `docs/artifacts/99-verify.md` 가 전 항목 통과다
- 최종 리포트: iteration 횟수, 루브릭 점수, (있다면) 미해소 gap 목록을 사용자에게 보고
