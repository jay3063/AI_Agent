# Doxygen 방식 주석 (Python)

`CLAUDE.md`: "주석은 Doxygen 방식으로 작성하며, 20% 이상 작성해야 한다."

## 작성 형식

Python에서 Doxygen이 인식하는 특수 독스트링 표기(`"""!` 시작)를 사용하고, 표준 Doxygen 태그(`@brief`, `@param`, `@return`, `@throws`, 필요 시 `@pre`/`@post`)를 포함한다. 상세설계 단계(`detailed-design` 스킬)의 함수 계약(사전조건/사후조건/오류 계약)을 그대로 주석에 반영한다 — 계약과 주석이 어긋나면 안 된다.

```python
def calcOrderTotal(itemPrices: list[float], taxRate: float) -> float:
    """!
    @brief 품목 가격 목록과 세율로 주문 총액을 계산한다.
    @param itemPrices 품목별 가격 목록 (원, 0 이상)
    @param taxRate 적용 세율 (0.0 ~ 1.0)
    @return 세금이 포함된 주문 총액 (원)
    @throws ValueError itemPrices에 음수가 포함되거나 taxRate가 범위를 벗어난 경우
    @pre itemPrices의 모든 값은 0 이상이어야 한다.
    @post 반환값은 항상 0 이상이다.
    """
    if any(price < 0 for price in itemPrices):
        raise ValueError("itemPrices에 음수가 포함될 수 없다")
    subtotal = sum(itemPrices)
    return subtotal * (1 + taxRate)
```

- 클래스와 모듈에도 동일하게 `"""!` + `@brief` 설명을 붙인다.
- 단순 getter/setter처럼 자명한 함수도 `@brief` 한 줄은 반드시 작성한다(주석 비율 20% 기준을 채우기 위한 형식적 나열이 아니라, 실제로 계약을 설명하는 내용이어야 한다).
- 구현 세부사항 중 비직관적인 부분(왜 이 알고리즘/자료구조를 선택했는지, 특정 예외 처리가 필요한 이유)은 함수 본문 내 `#` 라인 주석으로 보강한다. 이 라인 주석도 `radon raw`의 `Comments` 지표에 집계되어 20% 비율 계산에 포함된다.

## 20% 비율과의 관계

주석 비율 측정 방법과 공식은 `references/quality-metrics-and-tools.md`의 3절을 따른다(`(Comments + Multi) / LOC ≥ 0.20`). 이 문서의 태그 형식을 따르면 자연스럽게 주석 비율을 충족하는 경우가 많지만, **비율을 채우기 위해 의미 없는 문장을 반복하지 않는다.** 비율이 부족하면 실제로 빠진 계약 설명(예외 조건, 경계값, 왜 이 로직인지)을 찾아 보강한다.
