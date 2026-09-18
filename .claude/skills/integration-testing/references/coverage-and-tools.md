# 구조적 커버리지 측정: 함수 커버리지 / Call 커버리지 (100%, 필수)

ISO 26262-6은 소프트웨어 아키텍처 수준의 구조적 커버리지 지표로 **함수 커버리지(Function Coverage)** 와 **Call 커버리지(Call Coverage)** 를 요구한다. 이 프로젝트에서는 두 지표를 **100%** 달성해야 하며, 반드시 실제 도구 실행 결과로 측정한다(추측/육안 판단 금지).

## 0. 테스트 베이시스 (무엇을 기준으로 시험하는가)

통합시험 케이스는 다음 두 산출물을 **테스트 베이시스**로 도출한다. 임의로 통합시험 케이스를 만들지 않는다.

1. **아키텍처 설계서의 인터페이스 명세**(`architecture-design` 스킬 산출물 6장) — 각 인터페이스(제공자/사용자, 시그니처, 데이터 계약, 사전/사후조건, 오류 계약)가 시험 대상의 전체 목록이 된다. 모든 내부/외부 인터페이스가 최소 1개 이상의 시험 케이스로 커버되어야 한다.
2. **아키텍처 설계서의 통합 순서**(`architecture-design` 스킬 산출물 11장 통합 전략/순서 테이블) — 통합시험은 이 순서를 **그대로** 따른다. 순서상 앞선 통합 항목이 통과하기 전에는 뒤 항목을 시험하지 않는다(필요한 스텁/드라이버는 순서 테이블에 정의된 대로 사용한다). 순서를 임의로 재배열하지 않으며, 재배열이 필요하면 근거와 함께 불일치로 보고한다.

상세설계(`detailed-design` 스킬 산출물)의 함수 계약/호출관계는 인터페이스 뒤편의 실제 구현 세부(어떤 함수가 어떤 함수를 호출하는지)를 제공하며, 커버리지 측정 시 "선언된 호출 엣지" 집합의 근거가 된다.

## 1. 함수 커버리지 (Function Coverage) = 100%

**정의**: 아키텍처/상세설계에 정의된 모든 함수·메서드가 통합시험 실행 중 최소 1회 진입(entry)해야 한다.

**측정 절차 (오픈소스 도구 `coverage.py` 사용)**

```bash
pip install coverage
coverage run -m unittest discover -s <통합시험 디렉터리>
coverage json -o coverage.json
```

`coverage.py`는 라인 단위 커버리지만 보고하므로, 함수 단위 판정은 아래와 같이 표준 라이브러리 `ast`로 각 함수의 라인 범위를 추출한 뒤 `coverage.json`의 실행된 라인과 교차 검증한다.

```python
import ast
import json

def listFunctionRanges(sourcePath):
    """!
    @brief 소스 파일의 모든 함수/메서드 정의를 (이름, 시작줄, 끝줄)로 나열한다.
    @param sourcePath 분석할 Python 소스 파일 경로
    @return (함수명, 시작줄, 끝줄) 튜플 목록
    """
    tree = ast.parse(open(sourcePath, encoding="utf-8").read())
    ranges = []
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            ranges.append((node.name, node.lineno, node.end_lineno))
    return ranges

def calcFunctionCoverage(sourcePath, coverageJsonPath):
    """!
    @brief coverage.json의 실행 라인과 함수 범위를 교차 검증해 미실행 함수를 반환한다.
    @param sourcePath 대상 소스 파일 경로
    @param coverageJsonPath coverage.py의 JSON 리포트 경로
    @return 미커버 함수명 목록(비어 있으면 100%)
    """
    with open(coverageJsonPath, encoding="utf-8") as coverageFile:
        report = json.load(coverageFile)
    executedLines = set(report["files"][sourcePath]["executed_lines"])
    uncoveredFuncs = [
        name for name, start, end in listFunctionRanges(sourcePath)
        if not (executedLines & set(range(start, end + 1)))
    ]
    return uncoveredFuncs
```

- 미커버 함수가 하나라도 있으면 **함수 커버리지 100% 미달**이다. 해당 함수를 진입시키는 통합시험 케이스를 추가하고 재측정한다.
- 아키텍처/상세설계에 존재하지만 소스에 구현되지 않은 함수, 혹은 소스에는 있지만 어떤 인터페이스/요구사항에도 매핑되지 않는 함수(고아 함수)는 별도로 보고한다.

## 2. Call 커버리지 (Call Coverage) = 100%

**정의**: 아키텍처 설계서(5장 정적 의존성)와 상세설계(3장 호출관계)에 **선언된 모든 호출 엣지(caller → callee)** 가 통합시험 실행 중 최소 1회 실제로 호출되어야 한다. 함수 커버리지(함수가 진입되었는가)와 달리, Call 커버리지는 "누가 누구를 호출했는가"라는 관계 자체를 검증한다 — 통합시험의 핵심 목적(컴포넌트 간 상호작용 검증)과 직접 연결된다.

**측정 절차 (표준 라이브러리 `trace` 모듈 사용)**

```python
import trace

def runIntegrationSuiteWithCallTrace(runSuiteFunc):
    """!
    @brief 통합시험 스위트를 실행하며 실제 caller→callee 호출 관계를 기록한다.
    @param runSuiteFunc 통합시험 스위트를 실행하는 콜러블(예: unittest 러너 호출)
    @return trace.Trace 결과 객체 (caller_relationships 조회 가능)
    """
    tracer = trace.Trace(count=False, trace=False, countcallers=True)
    tracer.runfunc(runSuiteFunc)
    return tracer.results()
```

`results().calledfuncs`/`caller` 관계(버전에 따라 `trace.CoverageResults`의 내부 구조 확인 필요 — `python -m trace --help`로 CLI 옵션도 함께 확인한다)에서 실제 실행된 `(caller, callee)` 쌍 집합을 추출한다.

**선언된 호출 엣지와 비교**

1. 아키텍처 5장(정적 의존성)과 상세설계 3장(상세 호출관계) 문서에서 "선언된 호출 엣지" 집합(Mermaid 그래프의 각 엣지)을 목록화한다.
2. 실행 중 실제로 관측된 호출 엣지 집합과 비교한다.
   - **선언되었으나 실행되지 않은 엣지** → Call 커버리지 갭. 그 엣지를 실행시키는 통합시험 케이스를 추가하고 재측정한다.
   - **실행되었으나 선언되지 않은 엣지** → 설계 문서와 구현의 불일치. 추측으로 무시하지 말고 별도로 보고한다(설계 갱신 또는 구현 수정 필요 여부를 사용자에게 확인).
3. 두 집합이 완전히 일치(선언된 엣지가 모두 실행됨)할 때 Call 커버리지 100%로 판정한다.

## 3. 보고 형식

| 지표 | 대상 수 | 커버된 수 | 커버리지 | 미커버 목록 | 판정 |
|---|---|---|---|---|---|
| 함수 커버리지 | ... | ... | ...% | (있다면 함수명 나열) | Pass/Fail |
| Call 커버리지 | ... | ... | ...% | (있다면 엣지 나열: caller→callee) | Pass/Fail |

- 100% 미달 상태를 "거의 다 됐다"는 식으로 완료 보고하지 않는다. 갭을 메우는 시험 케이스를 추가하고 **재실행·재측정한 결과**까지 포함해야 완료로 본다.
- 두 지표 모두 도구 실행 원문(요약)을 근거로 인용한다.
