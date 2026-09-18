# 용어집 (Glossary) — 전자식 차일드락 제어 SW

> 이 용어집은 `requirements-analysis` 스킬의 일관성 관리 절차에 따라 유지된다. 신규 도메인 용어가 등장하면 이 파일에 추가하고, 기존 용어와 동의어를 혼용하지 않는다.

| 용어 | 정의 | 비고 |
|---|---|---|
| 차일드락(Child Lock) | 후석 좌/우 도어의 논리적 잠금/해제 상태를 제어하는 기능. 본 프로젝트는 SW 논리 결정까지만 다루며 도어 래치/구동기 HW는 범위 밖. | OEM 원문 3절 |
| LOCK / RELEASE | 출력(lock_left, lock_right)의 상태값 (OEM-IF-005). "잠금 상태"/"해제 상태"를 가리키는 **출력값**. | action(명령)과 구분할 것 |
| lock / unlock | 운전자 명령(OEM-IF-004)의 action 필드 값. 사용자가 요청한 **명령**. | 출력값 LOCK/RELEASE와 1:1 대응이 아님(다른 트리거에 의해 억제/무시될 수 있음) — SWR-004 참조 |
| 해제 | "unlock 명령" 또는 "RELEASE 출력"을 모두 가리킬 수 있는 일반 한국어 표현. 요구사항 문장에서는 반드시 "unlock 명령" 또는 "RELEASE 출력" 중 하나로 명시해 혼용하지 않는다. | 명확성 체크리스트 4항 관련 |
| side | left / right / all. 명령 또는 판단이 적용되는 대상 도어. | OEM-IF-004 |
| 접근위험(Approach Risk) | 후측방 접근위험 불리언 신호(rear_left/right_approach_risk). TRUE일 때 해당 도어 LOCK 강제 및 해제 억제. | OEM-SR-002 |
| Override | 접근위험 억제 후 10초 이내 동일 도어 동일 unlock 명령 재입력 시 성립하는 조건. 2026-09-18 사용자 확정(`SWE1-001` §8.2 결정2)에 따라 성립 시 해당 도어 출력을 실제로 RELEASE로 전환하고 override 상태·이유코드를 함께 제공한다. | OEM-FR-003, SWR-006 |
| crash_status | NONE/PENDING/CONFIRMED 3값 열거형. CONFIRMED는 SWR-022 우선순위 정책의 순위 1(최우선)로 강제 RELEASE를 유발(SWR-007). PENDING의 능동적 의미는 정의하지 않으며, PENDING 동안 다른 트리거는 정상 평가된다(SWR-008, 2026-09-18 확정). | OEM-IF-002 |
| sensor_fault | 정규화 입력의 고장 플래그. TRUE 시 "새 명령 미적용, 직전 확정 출력 유지, state=FAULT"(SWR-021). SWR-022 우선순위 정책의 순위 3으로 확정(2026-09-18)되어, crash_status=CONFIRMED(순위1)·화재/과열/성인탑승(순위2)이 동시에 유효한 경우에는 그 강제 RELEASE가 우선하고 sensor_fault의 출력유지 지시는 적용되지 않는다. | OEM-SR-004 |
| 트리거 우선순위 정책 | 크래시/화재등/센서고장/접근위험/ISOFIX/ignition-off/자동잠금/운전자명령 8개 트리거가 동시에 유효할 때 어떤 트리거의 출력 지시가 적용되는지를 정하는 SWR-022(순위 1~8). OEM 원문에는 전체 순서 근거가 없으며 2026-09-18 사용자 승인으로 정의된 정책이다. | `SWE1-001` §4.2.5, §8.2 |
| ISOFIX | 좌/우 ISOFIX 카시트 연결 감지 신호(isofix_left/right). TRUE 시 해당 도어 강제 LOCK. | OEM-SR-004(원문 목적), OEM-FR-006(수용기준) — 분류는 OEM-FR-006이 QM으로 지정한 것을 그대로 따름 |
| freshness | 입력이 정의된 주기 내에 갱신되었는지 여부. source_timestamp_s 기준 200ms 초과 시 미갱신으로 간주. | OEM-SR-003 |
| DEGRADED | 필수 입력의 freshness 위반이 감지된 상태. state 필드의 값. | OEM-SR-003 |
| FAULT | sensor_fault=TRUE로 확인된 상태. state 필드의 값. DEGRADED와 별개의 상태. | OEM-SR-004 |
| OFF | ignition_on=FALSE로 확인된 상태. state 필드의 값. | OEM-FR-007 |
| state | 시스템 표시 상태 필드. NORMAL/DEGRADED/FAULT/OFF 중 하나. FAULT/OFF와 LOCK/RELEASE 출력의 우선순위는 SWR-022(트리거 우선순위 정책)로 결정된다. 단, DEGRADED(SWR-013-A, freshness 위반)는 SWR-022의 8단계 표에 포함되지 않아 다른 트리거와의 동시발생 시 표시 우선순위가 아직 정의되지 않았다(`SWE1-001` §8.3 잔존-3, 비안전크리티컬·재확인 권장). | OEM-IF-006 |
| reason_code / priority_reason | 최근 결정의 원인을 나타내는 코드. OEM-IF-006에 따라 priority_reason은 reason_code로도 동일하게 매핑됨. | OEM-IF-006 |
| 결정 레코드(Decision Record) | 매 평가주기의 제어결정을 표현하는 메모리 내 레코드(허용 필드만 포함, PII/영상/음성 금지). | OEM-NFR-002 |
| PC/SIL | Software-in-the-Loop 형태의 PC 상 소프트웨어 전용 검증 환경. HIL/실차 제외. | OEM 원문 1.3절 |
| ASIL B (입력) | 가상 OEM이 이미 HARA/ASIL 도출을 완료해 공급자에게 할당한 안전 요구 등급. 본 프로젝트는 이 등급을 그대로 받아들이며 재도출하지 않음(ISO 26262 Part 3 범위 밖). | OEM 원문 2절 |
| QM | ASIL이 아닌 품질관리(Quality Management) 등급의 요구사항. | OEM 원문 5절 |

## 용어 사용 원칙

- 요구사항 문장에서 "명령(command)"은 항상 운전자/HMI 입력(OEM-IF-004)만을 가리키며, 자동 트리거(충돌/화재/ISOFIX/접근위험/자동잠금/ignition-off)의 판단 결과는 "명령"이 아니라 "트리거" 또는 "강제 조건"으로 표기한다. 이 구분이 흐려지면(예: OEM-SR-001의 "다른 명령보다 우선"이 자동 트리거까지 포함하는지) 확인 필요 섹션에 별도로 기록한다.
