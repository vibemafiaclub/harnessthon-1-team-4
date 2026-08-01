---
name: stage-verify-penpot
description: 파이프라인 종료 전 Penpot을 되읽어 화면·산출물이 실제로 남았는지 확인하고 docs/artifacts/99-verify.md로 만든다. 하나라도 실패하면 start는 완료를 선언하지 않는다
---

<!-- 담당자: 조장 (start 오케스트레이터와 한 몸인 최종 게이트) -->

# Verify — 진짜 남았는지 되읽는 최종 게이트

Evaluate(품질 루프)와 다르다. 이것은 **존재 검증**이다: 하네스가 "다 됐다"고 말하기 전에
Penpot과 `docs/artifacts/`를 다시 읽어 실제로 남아 있는지 확인한다. **읽기 전용.**

## 입력 (이것만 읽는다)

- 작업 Page 이름 (인자) — **없으면 즉시 중단하고 요구한다.** 추측하지 않는다
- `docs/artifacts/00-clarify.md` — 화면 목록
- `docs/artifacts/` 디렉토리 목록
- Penpot MCP — 읽기 호출만. **Page를 전환하지 않는다**

## 출력 (이것만 쓴다)

- `docs/artifacts/99-verify.md`

## 절차

1. Page 인자를 확인한다. 없으면 중단·보고.
2. **[검증 1]** 작업 Page에 board/frame이 **1개 이상** 있는가.
   (`pages.find(...)`로 Page 객체를 얻어 순회 — 전환 금지)
3. **[검증 2]** 00-clarify의 화면 목록 **전부**가 Page에 있는가.
   `04-screens-log.json`의 매핑으로 board를 찾아 대조하고, **누락 화면 목록**을 적는다.
4. **[검증 3]** 각 단계 산출물이 실제로 남아 있는가:
   `00-clarify.md`, `01-context.json`, `02-plan/tokens.json`, `02-plan/components.json`,
   `02-plan/layouts/` (화면 수만큼), `03-library-log.json`, `04-screens-log.json`,
   `05-eval-report.md`. 없는 파일 목록을 적는다.
5. 실패 항목마다 **담당 단계**(재실행 대상)를 명시한다.
6. 출력 파일을 쓴다.

## 출력 형식

```markdown
# 99-verify

## 종합: PASS | FAIL

| 검증 | 결과 | 상세 |
|---|---|---|
| 1. Page에 board ≥ 1 | PASS/FAIL | board N개 |
| 2. 화면 전부 존재 | PASS/FAIL | 누락: S03(화면명), … |
| 3. 산출물 전부 존재 | PASS/FAIL | 누락: 03-library-log.json, … |

## 실패 → 재실행 대상
| 실패 항목 | 담당 단계 |
```

## 금지

- **쓰기·수정 금지.** Penpot에 아무것도 만들지 않는다. Page 전환 금지.
- 확인 없이 PASS를 적지 않는다. 확인 실패(에러)는 FAIL로 기록하고 사유를 남긴다.
- 다른 단계의 출력 파일을 쓰지 않는다.
