# 명명 규칙: 3글자 이상, 낙타 표기법(camelCase)

`CLAUDE.md`: "함수명, 변수명은 3글자 이상 사용하고, 낙타 표기법을 활용한다."

## 규칙

- 함수명/변수명은 **3글자 이상**이어야 한다. `i`, `x`, `ok`처럼 2글자 이하인 식별자는 금지한다(단, 표준 관례상 `for i in range(...)`의 루프 카운터처럼 극히 짧은 스코프에 한정된 경우라도 이 프로젝트에서는 3글자 이상을 적용한다 — 예: `idx`, `row`).
- 함수명/변수명은 **낙타 표기법(lowerCamelCase)**을 사용한다: 첫 글자는 소문자, 이후 단어의 첫 글자는 대문자. 예: `calcTotalPrice`, `retryCount`.
- Python 표준 관례(PEP 8, snake_case)와 다르다는 점을 인지하고, 이 프로젝트에서는 **CLAUDE.md 지침을 우선**한다. 클래스명은 PascalCase(Python 일반 관례)를 유지해도 되지만, 함수/메서드/변수/매개변수는 lowerCamelCase를 따른다.
- 상수(모듈 전역 불변값)에 대해서는 프로젝트에서 별도 지시가 없는 한 UPPER_SNAKE_CASE 관례와 충돌할 수 있으므로, 상수 명명 규칙은 사용자에게 확인하거나 낙타 표기법을 일관되게 적용한다.

## 예시

| 잘못된 예 | 올바른 예 |
|---|---|
| `def f(x):` | `def calcSquare(inputVal):` |
| `n = 0` | `retryCount = 0` |
| `def get_user_by_id(id):` | `def getUserById(userId):` |
| `tmp`, `val`, `ok` (3글자 미만 또는 의미 없는 이름) | `tempBuffer`, `isValid` |

## pylint를 이용한 자동 점검 (참고)

`pylint`의 명명 규칙 검사(`invalid-name`, C0103)를 아래와 같이 정규식으로 설정하면 "3글자 이상 + lowerCamelCase"를 기계적으로 강제할 수 있다. `.pylintrc`(또는 `pyproject.toml`의 `[tool.pylint.basic]`)에 추가한다.

```ini
[BASIC]
function-rgx=[a-z][a-zA-Z0-9]{2,}$
variable-rgx=[a-z][a-zA-Z0-9]{2,}$
argument-rgx=[a-z][a-zA-Z0-9]{2,}$
```

- 정규식 `[a-z][a-zA-Z0-9]{2,}$`는 소문자로 시작하고 이후 2글자 이상(즉 총 3글자 이상)을 요구하여 3글자 미만 식별자를 걸러낸다. camelCase 자체(중간 대문자 사용)는 이 정규식으로 강제되지 않으므로, 코드 리뷰 시 사람이 함께 확인한다.
