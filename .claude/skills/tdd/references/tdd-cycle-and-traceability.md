# 추적성 보고 (함수ID ↔ 요구사항ID ↔ 테스트)

Red-Green-Refactor 절차 자체는 `SKILL.md` 본문을 따른다. 이 문서는 **완료 보고 시 남겨야 하는 추적성 표** 형식만 정의한다.

## 입력

- 상세설계 산출물(`detailed-design` 스킬 결과)의 함수 계약: 함수 ID, 함수명, 소속 모듈, 할당 요구사항 ID.
- 작성한 테스트: 클래스/메서드명, `references/test-documentation.md` 형식의 `@brief`/`@technique`/`@case`, 실행 결과.

## 명명 연계

테스트 클래스/메서드명은 대상 함수 ID/함수명과 연결되도록 짓는다(명명 규칙은 `references/naming-convention.md` 준수: 3글자 이상 + 낙타 표기법). 예: 함수 ID `FN-MOD03-012`, 함수명 `calcOrderTotal` → 테스트 클래스 `TestCalcOrderTotal`, 메서드 `testCalcOrderTotalWithValidInput`, `testCalcOrderTotalRejectsNegativePrice`.

## 추적성 표

구현이 끝나면 아래 표를 보고한다.

| 함수 ID | 함수명 | 소속 모듈(상세설계 2장 기준) | 할당 요구사항 ID | 테스트 클래스/메서드 | 케이스(긍정/부정) | 기법 | 테스트 결과 |
|---|---|---|---|---|---|---|---|
| FN-MOD03-012 | calcOrderTotal | MOD-03 | SW-REQ-041 | `TestCalcOrderTotal.testCalcOrderTotalAppliesTaxRateCorrectly` | 긍정 | 동등분할 | Pass |
| FN-MOD03-012 | calcOrderTotal | MOD-03 | SW-REQ-041 | `TestCalcOrderTotal.testCalcOrderTotalRejectsNegativePrice` | 부정 | 오류추정 | Pass |

- 함수 계약과 매핑되지 않는 코드(고아 코드), 테스트가 없는 공개 함수, 부정 케이스가 없는(오류 계약이 있음에도) 함수를 발견하면 별도로 보고한다.
- 이 표는 이후 SWE.4 단위시험 산출물(`TPL-SWE4-001/002`)과 SWE.5 통합시험 산출물의 입력이 되므로, 함수 ID/테스트 식별자를 그 문서에서도 재사용 가능하도록 일관되게 명명한다.
