# 테스트 함수 문서화 규칙 (Doxygen 목적/기법/긍정·부정 케이스)

모든 `unittest` 테스트 메서드는 프로덕션 코드(`references/doxygen-comment-style.md`)와 동일하게 Doxygen 형식(`"""!`)의 docstring을 가져야 하며, 다음 세 가지를 **반드시** 포함한다. 하나라도 빠지면 테스트 작성이 완료된 것으로 보지 않는다.

| 태그 | 내용 | 필수 여부 |
|---|---|---|
| `@brief` | 이 테스트가 무엇을 검증하는지, 어떤 프로덕션 코드 변경이 이 테스트를 실패시키는지 한 문장으로 설명 | 필수 |
| `@technique` | 이 테스트 케이스를 도출한 테스트 설계 기법 | 필수 |
| `@case` | 긍정(Positive) 또는 부정(Negative) 케이스 여부와 그 이유 | 필수 |

Doxygen 빌드 산출물에 두 태그를 보기 좋게 렌더링하려면 Doxyfile에 다음 별칭을 등록한다(선택, 문서화 관례 자체는 별칭 등록 여부와 무관하게 적용한다).

```
ALIASES += technique="@par 사용 기법:^^"
ALIASES += case="@par 케이스 구분:^^"
```

## 작성 형식

```python
def testCalcOrderTotalRejectsNegativePrice(self):
    """!
    @brief itemPrices에 음수가 포함되면 ValueError가 발생하는지 검증한다.
    @technique 오류추정(Error Guessing) - 계약 위반 입력값
    @case 부정(Negative) - 사전조건 위반 시 오류 계약(ValueError) 확인
    """
    with self.assertRaises(ValueError):
        calcOrderTotal([-1000.0], 0.1)
```

```python
def testCalcOrderTotalAppliesTaxRateCorrectly(self):
    """!
    @brief 세율이 적용된 정상 입력에 대해 올바른 총액을 반환하는지 검증한다.
    @technique 동등분할(Equivalence Partitioning) - 유효한 가격/세율 대표값
    @case 긍정(Positive) - 정상 경로가 손으로 계산한 기대값과 일치하는지 확인
    """
    self.assertEqual(calcOrderTotal([1000.0, 2000.0], 0.1), 3300.0)
```

## 테스트 설계 기법 어휘 (`@technique`에 사용)

상세설계 함수 계약(사전/사후조건, 오류 계약, 경계값)에서 케이스를 도출할 때 아래 기법 중 실제로 적용한 것을 명시한다. 임의의 이름을 짓지 말고 이 표에서 고르거나, 표에 없는 기법을 썼다면 그 기법의 표준 명칭을 조사해 기입한다.

| 기법 | 설명 | 적용 예 |
|---|---|---|
| 동등분할(Equivalence Partitioning) | 입력을 유효/무효 클래스로 나누고 각 클래스의 대표값으로 검증 | 정상 가격 범위 대표값 |
| 경계값분석(Boundary Value Analysis) | 유효 범위의 경계(최소/최대/경계-1/경계+1)를 검증 | 재시도 횟수 상한(3회) 전후 |
| 오류추정(Error Guessing) | 계약 위반, 잘못된 타입, null/빈 값 등 실패가 예상되는 입력을 검증 | 음수 가격, 빈 리스트 |
| 결정테이블 테스트(Decision Table Testing) | 여러 조건의 조합에 따른 기대 동작을 표로 도출해 검증 | 상세설계 7장 정책 의사결정표의 조합 |
| 상태전이 테스트(State Transition Testing) | 상태 다이어그램의 전이/가드/금지 전이를 검증 | 상세설계 8장 상태전이 상세 |
| 동시성/재진입성 테스트 | 동시 호출·재진입 시 계약이 유지되는지 검증 | 재진입 가능 함수의 병행 호출 |

## `@case` 판정 기준

- **긍정(Positive)**: 계약이 허용하는 유효한 입력/상태에 대해 정상 동작(정상 반환값, 정상 상태 전이)을 검증하는 케이스.
- **부정(Negative)**: 계약이 금지하는 입력/상태(사전조건 위반, 잘못된 타입, 경계 초과, 자원 실패 등)에 대해 오류 계약(예외/오류코드/거부 동작)이 지켜지는지 검증하는 케이스.
- 하나의 함수 계약에는 최소 1개의 긍정 케이스와, 오류 계약이 정의되어 있다면 최소 1개의 부정 케이스가 있어야 한다. 부정 케이스가 누락되면 "오류와 방어 동작"(상세설계 10장) 검증이 비어 있는 것으로 보고한다.
