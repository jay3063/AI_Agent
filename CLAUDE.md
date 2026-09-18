# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This repository is currently empty (a fresh git repo with no commits and no files). There is no build, lint, or test tooling to document yet.

## 소프트웨어 개발 정보

이 소프트웨어 개발은 다음의 항목을 이용해서 개발한다.

- Python 3.12를 이용해서 개발한다. (OEM-SWR-001 3절 "실행 환경: Python 3.12 PC/SIL" 기준)
- 코드 테스트는 unittest를 활용한다.
- 순환복잡도, 함수라인수 등 측정 지표는 오픈소스 도구를 이용한다.

## 프로젝트 개발 정책

### 개발 생명 주기

- 이 프로젝트는 **반드시** 분석, 설계, 구현, 테스트의 순서로 개발을 진행한다.
- 각 단계가 완료되었을 때, 지정된 템플릿을 이용한 산출물이 생성되어야 한다.

### 분석 지침
- 요구사항 분석 단계 수행은 requirements-analyst 서브에이전트가 담당한다.

### 아키텍처 설계 지침

- 아키텍처 설계 단계 수행은 architecture-designer 서브에이전트가 담당한다.

### 상세설계 지침

- 상세설계 단계 수행은 detailed-designer 서브에이전트가 담당한다.
- 상세설계는 아키텍처 설계가 정한 컴포넌트 경계·인터페이스·통합 순서를 재정의하지 않고, 그 안에서 모듈/함수 단위로 세분화한다.

### 구현 지침

- 구현 단계 수행은 coding 서브에이전트가 담당한다.
- TDD 방식으로 진행하고, TDD 스킬을 사용해야 한다.
- 다음의 품질 지표를 **반드시** 준수해야 한다.
  - 함수 라인수는 순수코드라인 50라인 이하여야 한다.
  - 함수 순환복잡도는 10 이하여야 한다.
  - 중복 코드는 7라인까지 허용한다.
  - 주석은 Doxygen 방식으로 작성하며, 20% 이상 작성해야 한다.
- 함수명, 변수명은 3글자 이상 사용하고, 낙타 표기법을 활용한다.

### 단위 테스트 지침

- 단위 테스트는 TDD로 대체한다.
- 단위 테스트는 Branch 커버리지 100%를 달성해야 한다.
- 테스트 성공률은 100%여야 한다.

### 통합 테스트 지침

- 통합 테스트는 integration-tester 가 수행한다.
- 테스트 성공률은 100%여야 한다.

### 시스템 테스트 지침

- 시스템 테스트는 sw-system-tester 가 수행한다.
- 테스트 성공률은 100%여야 한다.

## Git 브랜치 정책

AI 보조(바이브 코딩) 개발 + GitHub을 사용할 때 일반적으로 적용되는 트렁크 기반 정책을 따른다.

- `main`은 보호된 트렁크다. **직접 커밋/푸시하지 않는다.** 모든 변경은 브랜치를 만들어 작업한다.
- 브랜치 명명: `feature/<설명>`(기능 추가), `fix/<설명>`(버그 수정), `docs/<설명>`(문서), `chore/<설명>`(잡무/설정). 예: `feature/architecture-design-skill`.
- 모든 변경은 **Pull Request를 통해서만** `main`에 병합한다.
- PR은 `.github/workflows/ci.yml`의 CI Action(품질 게이트 + 단위 테스트, "지속적 통합/지속적 테스트")이 **통과해야만** 병합 가능하다 — 브랜치 보호 규칙의 필수 상태 검사(required status check)로 설정한다.
- 리뷰어가 없는 1인 개발 특성상 승인(approval) 리뷰는 필수로 강제하지 않되, PR 기반 워크플로 자체는 강제한다.
- 병합은 Squash merge를 기본으로 하고, 병합 후 브랜치는 삭제한다. `main`에 대한 force-push와 브랜치 삭제는 금지한다.
- 커밋 메시지는 `<type>: <설명>`(예: `feat:`, `fix:`, `docs:`, `chore:`) 형식을 권장한다.
- 브랜치 보호 규칙 자체(필수 상태 검사, force-push 금지 등)는 GitHub 저장소 설정(Settings → Branches, 또는 `gh api repos/{owner}/{repo}/branches/main/protection`)에서 적용한다 — 이 저장소의 파일만으로는 강제되지 않으므로, 저장소 관리자가 최초 1회 설정해야 한다.

## CI (GitHub Actions)

- `.github/workflows/ci.yml`이 `main`을 대상으로 하는 모든 PR(및 `main`에 대한 push)에서 실행된다.
- 실행 내용(구현/단위 테스트 지침을 그대로 자동화): `unittest` 전체 실행(테스트 성공률 100%), `coverage`로 Branch 커버리지 100% 검증, `lizard`로 함수 순수코드라인/순환복잡도, `pylint`(`.pylintrc`)로 중복 코드/명명 규칙, `radon raw`로 Doxygen 주석 비율(≥20%) 검증.
- 추적 중인 `.py` 파일이 아직 없으면 이 게이트들은 통과(no-op)로 처리되고, 소스가 추가되는 순간부터 자동으로 적용된다.
- 통합시험(SWE.5)·시스템/자격시험(SWE.6)의 커버리지·성공률 게이트는 아키텍처/요구사항 문서를 테스트 베이시스로 삼는 에이전트 주도 작업이므로 이 범용 CI Action의 범위에 포함하지 않는다 — 각각 `integration-tester`, `sw-system-tester` 실행 시 별도로 검증한다.
