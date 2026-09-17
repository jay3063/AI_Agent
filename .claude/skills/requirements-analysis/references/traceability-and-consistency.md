# 추적성 및 일관성 관리 방안

## ID 체계

| Prefix | 의미 |
|---|---|
| `SG-xxx` | Safety Goal (ISO 26262) |
| `STK-REQ-xxx` | Stakeholder Requirement |
| `SYS-REQ-xxx` | System Requirement (A-SPICE SYS.2) — 기능/비기능 모두 이 prefix, 유형 필드로 구분 |
| `FSR-xxx` | Functional Safety Requirement |
| `TSR-xxx` | Technical Safety Requirement |
| `SW-REQ-xxx` | Software Requirement (A-SPICE SWE.1) |
| `NFR-<특성약어>-xxx` | 비기능 요구사항 (예: `NFR-SEC-003`, `NFR-PERF-011`) — ISO 25010 특성 약어: FUNC/PERF/COMPAT/USE/REL/SEC/MAINT/PORT |
| `ARCH-xxx` | 아키텍처 요소 (다운스트림 링크용) |
| `TC-xxx` | 테스트 케이스 (다운스트림 링크용) |

xxx는 3자리 이상 순번(001, 002, ...)이며, 한번 부여된 ID는 요구사항이 삭제되어도 재사용하지 않는다(이력 보존).

## 양방향 추적 유지 절차

1. 요구사항 생성/수정 시 해당 레코드의 Upstream/Downstream 필드를 채운다.
2. 같은 작업 안에서 `traceability-matrix.md`(산출물 디렉터리 내)를 아래 형식으로 갱신한다.

| ID | 레벨 | Upstream | Downstream | 검증방법/TC | 상태 |
|---|---|---|---|---|---|
| SYS-REQ-004 | System | STK-REQ-002 | SW-REQ-012, SW-REQ-013 | TC-045 | Draft |

3. 상위가 없는 최상위 요구사항(Stakeholder)이나 하위가 아직 없는 최신 요구사항은 필드를 비우지 말고 "TBD" 또는 "N/A(최상위)"로 명시한다.
4. 요구사항을 삭제/폐기할 때는 매트릭스에서 행을 지우지 않고 상태를 "Deprecated"로 표기해 이력을 남긴다.
5. 매트릭스 갱신 없이 요구사항 문서만 수정하는 것을 금지한다 — 두 산출물은 항상 같은 커밋/작업 단위로 함께 갱신한다.

## 일관성 관리 절차

1. `glossary.md`(용어집)를 유지한다. 새로운 도메인 용어가 등장하면 정의를 추가하고, 기존 용어와 동의어를 쓰지 않는다.
2. 요구사항 추가/수정 시 다음을 스캔한다.
   - **모순**: 동일 조건에서 서로 다른 결과를 요구하는 두 요구사항 쌍
   - **중복**: 표현은 다르지만 동일한 요구를 담은 두 요구사항 쌍
   - **용어 불일치**: 같은 개념을 다른 단어로 지칭
3. 발견 사항은 "일관성 점검 결과" 섹션에 `[문제유형] REQ-ID-A ↔ REQ-ID-B: 설명` 형식으로 기록한다. 문제가 없으면 "발견된 문제 없음"을 명시한다 (섹션 자체를 생략하지 않는다).
