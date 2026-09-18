---
name: tdd
description: Red-Green-Refactor 기반 TDD 방법론(obra/superpowers의 test-driven-development 스킬을 Python/unittest 환경 및 CLAUDE.md 구현 지침에 맞게 각색). 실패하는 테스트를 먼저 작성하고 최소 구현으로 통과시킨 뒤 리팩토링하며, 테스트 함수에는 Doxygen 형식으로 목적·기법·긍정/부정 케이스를 기록하고, 함수 순수코드라인/순환복잡도/중복코드/주석비율/명명규칙을 오픈소스 도구로 실측 검증한다. 코드 구현/구현 단계 작업을 요청받을 때 사용.
---

# TDD 구현 스킬

> 이 스킬은 [obra/superpowers](https://github.com/obra/superpowers)의 `test-driven-development` 스킬(MIT License)을 Python 3.14 / `unittest` 환경과 `CLAUDE.md` 구현 지침에 맞게 각색한 것이다. 원본의 Iron Law·Red-Green-Refactor·Rationalizations·Red Flags 구조를 유지하고, 이 프로젝트 고유의 품질 게이트(순수코드라인/순환복잡도/중복코드/Doxygen 주석비율/명명규칙)와 테스트 문서화 규칙(기법·긍정/부정 케이스)을 추가로 결합했다.

## 0. 입력 확인 (최우선)

- 구현 대상의 상세설계 산출물(`detailed-design` 스킬 결과: 함수 계약 — 함수 ID, 시그니처, 사전/사후조건, 오류 계약, 할당 요구사항 ID)을 실제로 확인한다(`Glob`/`Grep`/`Read`). 없으면 추측해서 구현하지 않고 사용자에게 알린다.
- Python 3.14 문법을 기준으로, 테스트는 표준 라이브러리 `unittest`만 사용한다.

## 핵심 원칙 (Iron Law)

```
실패하는 테스트 없이 프로덕션 코드를 작성하지 않는다.
```

- 테스트보다 먼저 구현 코드를 작성했다면 **삭제하고 다시 시작한다.** "참고용으로 남긴다", "테스트를 작성하면서 조금씩 맞춘다"는 모두 금지 — 삭제는 삭제다.
- 테스트를 보지 않고 통과하는지 확인하지 않았다면, 그 테스트가 올바른 것을 검증하는지 알 수 없다.
- **규칙의 문구를 어기는 것은 규칙의 정신을 어기는 것과 같다.** "이번만 TDD를 건너뛴다"는 생각이 들면 멈춘다 — 그것은 정당화(rationalization)다.

### 언제 적용하는가

- **항상 적용**: 신규 기능, 버그 수정, 리팩토링, 동작 변경
- **예외(사용자에게 먼저 확인)**: 버리는 프로토타입, 생성된 코드, 단순 설정 파일

## Red → Green → Refactor

```
[RED: 실패하는 테스트 작성] → [실패 확인(필수)] → [GREEN: 최소 구현] → [통과 확인(필수)] → [REFACTOR: 정리 + 품질 게이트] → 다음 테스트
```

### 1. RED — 실패하는 테스트를 먼저 작성

1. 대상 함수 계약(상세설계의 함수 ID) 하나를 선택해 하나의 행동(behavior)만 검증하는 `unittest.TestCase` 메서드를 작성한다.
2. **모든 테스트 메서드에는 Doxygen 형식 docstring으로 다음을 반드시 기록한다** (`references/test-documentation.md` 형식 준수):
   - `@brief` — 이 테스트가 무엇을 검증하는지(어떤 프로덕션 코드 변경이 이 테스트를 실패하게 만드는지 설명 가능해야 한다)
   - `@technique` — 사용한 테스트 설계 기법(동등분할/경계값분석/오류추정/결정테이블/상태전이 테스트 등)
   - `@case` — 긍정(Positive) 또는 부정(Negative) 케이스 여부와 그 이유

```python
def testCalcOrderTotalRejectsNegativePrice(self):
    """!
    @brief itemPrices에 음수가 포함되면 ValueError가 발생하는지 검증한다.
    @technique 오류추정(Error Guessing) - 사전조건 위반 입력
    @case 부정(Negative) - 사전조건 위반 시 오류 계약(ValueError) 확인
    """
    with self.assertRaises(ValueError):
        calcOrderTotal([-1000.0], 0.1)
```

3. **좋은 예 / 나쁜 예**

<Good>

```python
def testRetryOperationSucceedsOnThirdAttempt(self):
    """!
    @brief 처음 두 번 실패하고 세 번째에 성공하면 그 결과를 반환하는지 검증한다.
    @technique 경계값분석(Boundary Value Analysis) - 재시도 허용 한계(3회) 경계
    @case 긍정(Positive) - 재시도 정책이 성공 케이스를 정상 반환하는지 확인
    """
    attempts = {"count": 0}

    def operation():
        attempts["count"] += 1
        if attempts["count"] < 3:
            raise RuntimeError("fail")
        return "success"

    result = retryOperation(operation)

    self.assertEqual(result, "success")
    self.assertEqual(attempts["count"], 3)
```

이름이 명확하고, 실제 코드(더블/모킹 없이)를 검증하며, 한 가지 행동만 다룬다. (`references/writing-good-tests.md` 참고)

</Good>

<Bad>

```python
def testRetryWorks(self):
    mockOperation = MagicMock(side_effect=[RuntimeError(), RuntimeError(), "success"])
    retryOperation(mockOperation)
    self.assertEqual(mockOperation.call_count, 3)
```

이름이 모호하고, 목(mock)의 호출 횟수만 검증한다 — 실제 동작이 아니라 목의 존재를 테스트한다.

</Bad>

### 2. Verify RED — 실패를 눈으로 확인 (필수, 절대 건너뛰지 않는다)

```bash
python -m unittest <테스트 모듈 또는 클래스> -v
```

확인해야 할 것:
- 테스트가 **실패**하는가(에러가 아니라 실패인가)
- 실패 메시지가 예상한 이유(기능 미구현)인가, 오타/설정 오류가 아닌가

- **테스트가 통과해버린다면?** 이미 존재하는 동작을 테스트한 것이다 — 테스트를 다시 설계한다.
- **테스트가 에러(예외/오타)를 낸다면?** 에러를 고치고, 올바른 이유로 실패할 때까지 다시 실행한다.

### 3. GREEN — 통과시키는 최소 구현

계약을 만족시키는 데 필요한 만큼만 구현한다(YAGNI). 테스트에 없는 옵션/기능을 미리 추가하지 않는다.

### 4. Verify GREEN — 통과를 눈으로 확인 (필수)

```bash
python -m unittest <테스트 모듈 또는 클래스> -v
```

확인해야 할 것: 대상 테스트 통과, 기존 테스트 전체 통과, 출력에 경고/오류 없음.

- **테스트가 계속 실패한다면?** 코드를 고친다 — 테스트를 손대서 통과시키지 않는다.
- **다른 테스트가 깨졌다면?** 즉시 원인을 고친다.

### 5. REFACTOR — 정리 + 품질 게이트 (필수)

테스트가 Green을 유지하는 상태에서 중복 제거, 이름 개선, 함수 분리를 수행한다. 동작(behavior)을 추가하지 않는다.

이후 `references/quality-metrics-and-tools.md`를 따라 오픈소스 도구로 아래 **CLAUDE.md 품질 게이트**를 실측한다. 미달 항목은 재설계/리팩토링 후 **재측정**해서 통과시킨다 — 측정 없이 "기준 충족"이라고 서술하지 않는다.

| 지표 | 기준 | 도구 |
|---|---|---|
| 함수 순수코드라인(NLOC) | ≤ 50 | `lizard -L 50` |
| 순환복잡도(CCN) | ≤ 10 | `lizard -C 10` (보조: `radon cc`) |
| 중복 코드 | ≤ 7줄 | `pylint --enable=duplicate-code --min-similarity-lines=8` |
| Doxygen 주석 비율 | ≥ 20% | `radon raw`의 `(Comments + Multi) / LOC` |
| 함수/변수명 | 3글자 이상 + 낙타 표기법 | `references/naming-convention.md` |

프로덕션 코드의 Doxygen 주석 형식은 `references/doxygen-comment-style.md`를, 테스트 함수의 Doxygen 주석(목적/기법/케이스)은 `references/test-documentation.md`를 따른다.

### 6. 반복

다음 행동에 대해 1~5를 반복한다.

## 좋은 테스트란

| 기준 | 좋음 | 나쁨 |
|---|---|---|
| 최소성 | 한 가지만 검증. 이름에 "and"가 필요하면 나눈다 | `testValidatesEmailAndDomainAndWhitespace` |
| 명확성 | 이름이 행동을 설명한다 | `test1`, `testCase2` |
| 의도 표현 | 원하는 API를 보여준다 | 코드가 뭘 해야 하는지 가린다 |

테스트를 작성/수정할 때는 항상 `references/writing-good-tests.md`를 함께 확인한다: 어떤 프로덕션 변경이 이 테스트를 실패시키는지 먼저 이름 붙이기, 실제 동작에 대해서만 단정하기(목 자체를 검증하지 않기), 테스트 전용 코드는 테스트 유틸리티에만 두기, 목으로 대체하기 전에 의존성의 부작용을 파악하기.

## 흔한 자기 합리화

| 변명 | 실제로는 |
|---|---|
| "너무 단순해서 테스트할 필요 없다" | 단순한 코드도 깨진다. 테스트는 30초면 된다. |
| "나중에 테스트 쓰겠다" | 나중에 쓴 테스트는 바로 통과한다 — 아무것도 증명하지 못한다. 실패를 본 적이 없으므로 버그를 잡을 수 있다는 증거가 없다. |
| "사후 테스트도 목적은 같다(형식보다 정신)" | 사후 테스트는 "이게 뭘 하나"에 답하고, 사전 테스트는 "이게 뭘 해야 하나"에 답한다. 이미 짠 코드에 편향된다. |
| "이미 수동으로 확인했다" | 수동 테스트는 기록이 없고 재실행이 안 되며 압박 속에서 케이스를 빠뜨리기 쉽다. |
| "지금까지 쓴 시간이 아깝다" | 매몰비용 오류다. 신뢰 못 할 코드를 유지하는 것이 진짜 낭비다. |
| "참고용으로 남기고 테스트부터 쓰겠다" | 결국 그 코드에 맞춰 테스트를 쓰게 된다 — 그건 사후 테스트다. 삭제는 삭제다. |
| "테스트하기 어려운 건 설계가 불명확해서다" | 테스트가 하는 말을 들어라. 테스트하기 어려우면 쓰기도 어렵다. |
| "TDD가 느리게 만든다" | TDD가 실용적인 길이다 — 커밋 전에 버그를 잡고 회귀를 막는다. |

## 위험 신호 — 멈추고 다시 시작

- 테스트보다 먼저 구현 코드를 작성함
- 테스트가 바로 통과함(실패를 본 적 없음)
- 왜 실패했는지 설명할 수 없음
- "일단 이번만" 이라는 생각
- "참고용으로 남긴다", "기존 코드를 적용하며 테스트를 쓴다"

**위 신호가 보이면: 코드를 삭제하고 TDD로 다시 시작한다.**

## 완료 전 점검 체크리스트

- [ ] 새/변경된 모든 함수에 테스트가 있는가
- [ ] 각 테스트가 구현 전에 실패하는 것을 직접 확인했는가(Verify RED)
- [ ] 실패 이유가 기대한 것(기능 미구현)이었는가(오타/설정 오류 아님)
- [ ] 각 테스트를 통과시키기 위해 최소한의 구현만 했는가
- [ ] 모든 테스트가 통과하는가(Verify GREEN), 출력에 경고/오류가 없는가
- [ ] 테스트가 실제 동작을 검증하는가(불가피한 경우가 아니면 목 사용 안 함)
- [ ] 경계값/오류 케이스가 포함되었는가
- [ ] 모든 테스트 메서드에 Doxygen `@brief`/`@technique`/`@case`가 작성되었는가(`references/test-documentation.md`)
- [ ] CLAUDE.md 품질 게이트(NLOC/CCN/중복/주석비율/명명규칙)를 도구로 실측했는가
- [ ] 함수ID ↔ 요구사항ID ↔ 테스트 추적성 표를 작성했는가(`references/tdd-cycle-and-traceability.md`)

체크박스를 모두 채울 수 없다면 TDD를 건너뛴 것이다 — 처음부터 다시 한다.

## 막혔을 때

| 문제 | 해결 |
|---|---|
| 테스트 방법을 모르겠다 | 원하는 API를 먼저 적어보고, 단정문부터 작성한다. 필요하면 사용자에게 확인한다. |
| 테스트가 너무 복잡하다 | 설계가 너무 복잡한 것이다. 인터페이스를 단순화한다. |
| 모든 걸 모킹해야 한다 | 코드가 너무 결합되어 있다. 의존성 주입을 사용한다. |
| 테스트 준비(setup)가 방대하다 | 헬퍼로 추출한다. 여전히 복잡하면 설계를 단순화한다. |

## 디버깅 통합

버그를 찾았는가? 그 버그를 재현하는 실패 테스트를 먼저 작성하고 TDD 절차를 따른다. 테스트 없이 버그를 고치지 않는다.

## 최종 규칙

```
프로덕션 코드 → 그 코드를 실패시킨 테스트가 먼저 존재해야 한다
그렇지 않으면 → TDD가 아니다
```

사용자의 명시적 허락 없이는 예외를 두지 않는다.

## 추적성

구현이 끝나면 `references/tdd-cycle-and-traceability.md`의 표 형식으로 함수 ID ↔ 함수명 ↔ 소속 모듈 ↔ 할당 요구사항 ID ↔ 테스트 클래스/메서드 ↔ 테스트 결과를 보고한다.

## 참고자료 목차

- `references/writing-good-tests.md` — 테스트가 정직한지 판단하는 규칙(어떤 변경을 잡아내는가, 목을 검증하지 않기, Mutation Check 등)
- `references/test-documentation.md` — 테스트 함수 Doxygen 문서화 규칙(`@brief`/`@technique`/`@case`)과 테스트 설계 기법 어휘
- `references/quality-metrics-and-tools.md` — CLAUDE.md 품질 게이트 측정 도구(lizard/radon/pylint)
- `references/naming-convention.md` — 3글자 이상 + 낙타 표기법
- `references/doxygen-comment-style.md` — 프로덕션 코드 Doxygen 주석 형식
- `references/tdd-cycle-and-traceability.md` — 추적성 표 형식
