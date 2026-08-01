---
name: stage-6-evaluate
description: 화면 스크린샷과 Clarify 체크리스트를 대조하고 채점 A 루브릭으로 자체 채점해 합격 또는 gap 리포트를 docs/artifacts/05-eval-report.md로 만든다
---

<!-- 담당자: 미정 (팀 sync에서 확정) — 담당자만 이 파일을 고칩니다 -->

# 6. Evaluate — 시각 검증 · 충족도 검증

이 단계는 Penpot을 **읽기만** 한다 (export 포함). 고치는 것은 Generate의 일이다 —
여기서는 **무엇이 왜 부족한지**를 다음 루프가 바로 쓸 수 있는 형태로 적는다.

## 입력 (이것만 읽는다)

- `docs/artifacts/00-clarify.md` — 체크리스트 (PRD 해석의 단일 출처. PRD 원문을 다시
  해석하지 않는다)
- `docs/artifacts/04-screens-log.json` — 화면·노드 ID 매핑
- 작업 Page 이름 (인자) — 읽기 전용. **Page를 전환하지 않는다**
- 현재 iteration 번호 (인자)

## 출력 (이것만 쓴다)

- `docs/artifacts/05-eval-report.md`
- 스크린샷 PNG는 `docs/artifacts/eval-shots/{iteration}/S{nn}.png`에 저장

## 절차

1. 입력을 읽는다. 없으면 **즉시 중단하고 무엇이 없는지 보고한다.**
2. screens-log의 board마다 `export_shape`로 PNG를 찍는다.
   **빈 영역이 나오면 없다고 판단하기 전에 재-export 한 번** (레이아웃 안정 전에 찍히면
   빈 영역이 나온다). 재-export에도 비면 그때 gap으로 기록한다.
3. **충족도 검증**: 체크리스트 전 항목을 순회한다.
   - `presence` → 대상 화면 스크린샷·노드에서 실제 확인
   - `absence` → 있으면 안 되는 것이 없는지 확인
   - `global` → 전 화면에 걸쳐 확인
   - screens-log에 없는 화면 id(만들어지지 않은 화면)는 그 자체로 gap이다.
4. **시각 검증 + 루브릭 자체 채점** (채점 A, 각 1~5점):
   레이아웃·정렬 / 타이포·컬러 / 완성도·디테일 / PRD 충족도.
   3점 이하 항목은 반드시 구체적 gap으로 환원한다 ("무엇이 어느 화면에서 어떻게").
5. **판정**: 체크리스트 전 항목 통과 && 루브릭 전 항목 4점 이상 → `합격`.
   아니면 `불합격` + gap 표.
6. 출력 파일을 쓴다.

## 출력 형식

```markdown
# 05-eval-report (iteration N)

## 판정: 합격 | 불합격

## 루브릭 자체 채점
| 항목 | 점수 | 근거 |

## 체크리스트 대조
| 체크 id | 결과 (pass/fail) | 확인 방법 |

## gap 목록 (불합격 시 — 다음 루프의 입력)
| gap id | 체크 id | 화면 id | 대상 노드 ID | 무엇이 부족한가 | 담당 단계 (stage-3/4/5) |
```

## 금지

- **수정 금지.** 이 단계는 Penpot에 쓰지 않는다. Page 전환도 하지 않는다.
- PRD 원문을 재해석하지 않는다 — 체크리스트에 없는 요구를 새로 만들지 않는다.
- 입력에 없는 값을 지어내지 않는다. 특정 PRD 전용 하드코딩 금지.
- 다른 단계의 출력 파일을 쓰지 않는다.
