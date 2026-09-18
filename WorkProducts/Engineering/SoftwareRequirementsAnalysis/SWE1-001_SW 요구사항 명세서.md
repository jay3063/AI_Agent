# SWE1-001 SW 요구사항 명세서 — 전자식 차일드락 제어 SW

## 문서 통제

| 항목 | 내용 |
|---|---|
| 문서 ID / 명칭 | SWE1-001 / SW 요구사항 명세서 — 전자식 차일드락 제어 SW |
| 적용 템플릿 | TPL-SWE1-001_SW 요구사항 명세서 템플릿.docx (섹션 1~12 구조를 따름) |
| 버전 / 베이스라인 | Rev 0.2 / BL-OEM-1.0 (OEM 입력 베이스라인 불변, SW 요구사항만 개정) |
| 작성자 | 요구사항분석 에이전트 |
| 검토 요청 대상 | jay.kim3063@gmail.com |
| 승인자 | 대기 — 가상 OEM-A 역할을 겸임하는 사용자(jay.kim3063@gmail.com) |
| 작성 상태 | 전체 SWR 작성 완료(Draft), 검토 대기. 상세 승인 이력은 §8.2 참조 |

**교육용 시나리오 고지 (OEM-SWR-001 원문 2절 인용)**: 본 문서는 SW 품질교육을 위한 가상 OEM-A 입력(`OEM_Sample/OEM-SWR-001_OEM SW 요구사항 사양서.docx`, Rev 1.0, BL-OEM-1.0)에 근거한 교육용 산출물이며, 실제 제작사의 사양을 나타내지 않는다. 공급자(본 프로젝트)는 OEM이 입력으로 제공한 ASIL B 안전 분류를 그대로 받아들이며, HARA/ASIL 결정 근거를 재구성하지 않는다. ISO 26262 Part 3(HARA/ASIL 도출), ECU/HW, HIL, 실차, 공식 심사·인증 활동은 본 프로젝트 범위 밖이다. 본 문서와 이를 생성한 방법론(스킬)은 실무 보조 도구이며 ISO 26262·Automotive SPICE 원문 표준을 대체하지 않는다.

## 개정 이력

| Rev | 일자 | 작성자 | 변경 내용 |
|---|---|---|---|
| 0.1 | 2026-09-18 | 요구사항분석 에이전트 | 최초 작성. OEM-SWR-001(Rev 1.0, BL-OEM-1.0) 기반 SWR-001~021 도출. "사용자 확인 필요" 8개 우선순위 충돌 쌍 및 4개 해석 항목 등록. 템플릿 미확정으로 임시 양식 사용. |
| 0.2 | 2026-09-18 | 요구사항분석 에이전트 | (1) 사용자(가상 OEM-A 역할 겸임)의 확정 결정 4건 반영: 트리거 간 8단계 우선순위 정책을 신규 SWR-022로 명문화, SWR-006 override의 실제 RELEASE 전환 확정, crash_status=PENDING 처리 명확화, SWR-019 확정. (2) 이에 따라 SWR-003/004/005/006/007/008/017/018/019/020/021 본문·상태 갱신. (3) 실제 공식 템플릿(TPL-SWE1-001) 구조로 문서 전체 재정렬, "확정 템플릿 미반영" 경고 제거. (4) §8.2에 해결된 사용자 확인 이력(감사기록)을 신설. |

---

## 1. 목적 및 적용범위

### 1.1 목적

본 문서는 가상 OEM-A 입력(OEM-SWR-001)의 System/Stakeholder 레벨 요구사항으로부터 SWE.1 절차에 따라 Software 레벨 요구사항(SWR-001~022)을 도출·명세한다.

### 1.2 적용범위

대한민국 판매용 2026년식 가상 OEM-A 승용차 후석 좌/우 전자식 차일드락 제어 SW. ASIL B로 할당된 SW 안전 요구와 QM 요구를 구분한다. 충돌·화재·과온·성인 탑승 강제해제·접근위험·ISOFIX 강제잠금·트리거 간 우선순위 결정을 포함한 PC/SIL 논리를 다룬다. HARA, ASIL 도출, ISO 26262 Part 3, ECU/HW, HIL, 실차, 인증 활동은 제외한다.

### 1.3 적용 경계

- SW-only PC/SIL 및 Web 검증만 다룬다. HIL, 실차, 타깃 ECU, 시스템/HW 개발, Part 3 HARA, 공식 심사·인증은 범위 밖이다(§10 참조).
- 프로젝트 제약: 영상·음성 원본과 개인식별정보를 입력·저장하지 않으며, 성인 존재 여부는 불리언 시험신호로만 표현한다(SWR-010/012에 직접 반영).
- 실행 언어 정책: 본 저장소 CLAUDE.md 및 커밋 이력(`fix: align Python version policy with OEM-SWR-001 (3.14 -> 3.12)`)에 따라 참조 구현/자동시험 환경은 Python 3.12이다(OEM 원문 3절 "실행 환경"과 일치).
- 용어집: `WorkProducts/Engineering/SoftwareRequirementsAnalysis/glossary.md` 참조. 모든 요구사항 문장은 이 용어집의 용어만 사용한다.

---

## 2. 요구사항 작성 및 판정 규칙

### 2.1 식별 및 상태 규칙

- **ID**: `SWR-NNN` (3자리 순번). 원자성 확보를 위해 하나의 상위 요구가 두 개 이상의 독립 검증 가능한 조건으로 분리되어야 하는 경우 `SWR-NNN-A`, `SWR-NNN-B` 접미사를 사용한다(예: SWR-013-A/013-B).
- **버전**: 문서 Rev와 함께 관리하며, 개별 SWR 단위 버전은 별도 부여하지 않고 개정 이력(위 표)에서 변경 시점을 추적한다.
- **상태값**: `Draft`(초안, 검토 대기 — 본문·검증기준·추적성 필드가 모두 채워진 완결 상태) / `Draft(부분)`(안전영향이 있는 미해결 의존성이 남아 완결로 볼 수 없는 상태 — 본 개정에서 전량 해소됨, §8.2) / `Review`(검토 진행 중, 미사용) / `Approved`(승인 완료, 미사용) / `Rejected`/`Superseded`(폐기·대체, 미사용). 본 개정 시점에는 모든 SWR이 `Draft`이다(§3).
- **우선순위(문서 관리용)**: `High`/`Medium`. 이는 검토·설계 착수 순서를 나타내는 **관리적 우선순위**이며, §4.2 SWR-022가 정의하는 **트리거 실행순위**(런타임에 여러 트리거가 동시 발생했을 때 어떤 트리거의 출력 지시가 적용되는지의 순서)와는 별개의 개념이다. 두 개념을 혼용하지 않는다(명확성 체크리스트 4항).
- **변경통제**: 요구사항 신규/수정은 개정 이력 표에 근거를 남기고, 사용자(OEM) 승인이 필요한 해석/결정은 §8.2에 감사기록으로 남긴다.

### 2.2 품질 판정 기준

- **명확성**: `references/clarity-checklist.md` 전 항목 통과. 모호한 수식어("적절히" 등), 복수 요구 결합 문장, 미정의 대명사를 허용하지 않는다.
- **원자성**: 하나의 SWR 본문은 하나의 검증 가능한 요구만 담는다. 다중 조건은 세분 ID(예: SWR-013-A/B)로 분리한다.
- **일관성**: §13(일관성 점검 결과) 및 `glossary.md`로 관리한다.
- **검증가능성**: 모든 SWR은 방법+합격기준을 갖는 실행 가능한 검증기준을 필수로 기재한다. 정성적 표현만 있는 검증방안은 반려·재작성한다.
- **추적성**: 모든 SWR은 Upstream/Downstream 필드를 필수로 기재하며, 아직 없는 Downstream은 "TBD"로 명시한다(필드 자체를 생략하지 않음).

---

## 3. 상태와 우선순위

| 상태값 | 의미 | 전이 조건 |
|---|---|---|
| Draft | 본문/근거/검증기준/추적성 필드가 모두 채워졌고, 안전영향이 있는 미해결 의존성이 없는 초안 | → Review(검토자가 검토 착수 시) |
| Draft(부분) | 안전영향이 있는 미해결 의존성(사용자 확인 대기 등)이 남아 완결로 볼 수 없는 초안 | 의존성 해소 시 → Draft (본 개정에서 전량 Draft로 전이됨) |
| Review | 검토 진행 중 (본 개정 시점 미사용) | → Approved 또는 Draft(반려) |
| Approved | 검토자·승인자 승인 완료 (본 개정 시점 미사용) | 변경 발생 시 → Draft |

승인 책임: 검토자는 jay.kim3063@gmail.com, 승인자는 가상 OEM-A 역할을 겸임하는 사용자이다(본 프로젝트에서는 동일인). 본 개정(Rev 0.2)은 §8.2에 기록된 4건의 사용자 결정을 반영한 초안이며, 문서 전체는 아직 `Draft` 상태로 정식 Review/Approve 이전이다.

---

## 4. 기능 및 안전 관련 SW 요구사항

### 4.1 기능 요구사항 (QM)

#### 4.1.1 OEM-FR-001 도출군 — 운전자 명령 처리 (SWR-001, 002, 004, 019)

**SWR-001 — 좌/우 개별 도어 명령 처리**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 side가 left 또는 right이고 action이 lock 또는 unlock인 유효한 운전자 명령(OEM-IF-004)을 수신하면, SWR-004에 정의된 우선 트리거가 해당 side에 적용되지 않는 한, 해당 side의 출력(lock_left 또는 lock_right)만 명령된 상태(LOCK 또는 RELEASE)로 설정해야 한다.
- 근거: OEM-FR-001 수용기준 "선택 출력에만 적용된다".
- Upstream: OEM-FR-001, OEM-IF-004 | Downstream: TBD
- 검증기준: 정지 상태, 상위 우선 트리거 비활성 조건에서 4개 source(physical_button/avn/voice/mobile_app) × 2 side × 2 action = 16개 조합을 PC/SIL 자동시험으로 주입, 다음 평가주기에 해당 side 출력만 전환되고 반대측 출력은 불변임을 검증(전수 16/16 통과가 합격기준).
- 검토 상태: Draft

**SWR-002 — 전체(ALL) 명령 처리**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 side가 all이고 action이 lock 또는 unlock인 유효한 운전자 명령을 수신하면, SWR-004의 우선 트리거가 적용되지 않는 도어에 한해 좌·우 출력을 모두 명령된 상태로 설정해야 한다.
- Upstream: OEM-FR-001, OEM-IF-004 | Downstream: TBD
- 검증기준: 4 source × 2 action = 8개 조합을 PC/SIL 자동시험으로 주입, 우선 트리거가 없는 도어의 좌·우 출력이 모두 전환됨을 검증(8/8 통과가 합격기준).
- 검토 상태: Draft

**SWR-004 — 상위 우선 트리거 적용 시 운전자 명령 게이팅 규칙**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 대상 도어에 SWR-007(충돌 우선 처리), SWR-017(화재/과열/성인탑승 강제해제), SWR-021(센서고장 유지), SWR-005/006(접근위험 억제/override), SWR-018(ISOFIX 강제잠금), SWR-020(ignition-off 강제해제) 또는 SWR-003(자동잠금) 중 SWR-022 우선순위 정책에 따라 그 평가주기에 유효한 트리거가 하나라도 있으면, SWR-001/002에 따른 운전자 명령의 결과를 그 도어에 적용하지 않아야 한다.
- 근거: OEM-FR-001 수용기준 "선택 출력에만 적용된다"는 표현과, SWR-003/005/006/007/017/018/020/021 각각의 "강제/우선" 문구가 운전자 명령보다 우선함을 논리적으로 요구함. 각 트리거 간, 그리고 트리거와 운전자 명령 간의 전체 순위는 SWR-022(신규, §4.2)로 명문화되었다(2026-09-18 사용자 확정, §8.2 결정 1).
- Upstream: OEM-FR-001 | Downstream: TBD
- 검증기준: SWR-003/005/007/017/018/020/021 각각을 단독 활성화한 상태에서 반대 방향의 운전자 명령(예: LOCK 강제 중 unlock 명령)을 주입, 명령이 적용되지 않고 강제 상태가 유지됨을 결정테이블 기법(7개 상위 조건 × 반대명령)으로 자동시험 검증. SWR-006 override가 성립하는 경우는 예외로 실제 RELEASE 전환이 일어남을 별도 케이스로 검증(SWR-006 참조).
- 검토 상태: Draft

**SWR-019 — 운전자 명령 필드 유효성 검증 및 거절**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 운전자 명령(OEM-IF-004: side, action, source)에서 필드 누락, 형식 오류, 또는 미등록 enum 값(side ∉ {left, right, all}; action ∉ {lock, unlock}; source ∉ {physical_button, avn, voice, mobile_app})을 검출하면, 해당 명령을 SWR-001/002의 평가 대상에서 거절하고 오류를 기록해야 한다.
- 근거: OEM-IF-004 오류 처리 조항("누락, 형식, 미등록 enum 거절")이 OEM-FR-001의 수용기준(정상입력만 규정)이나 기존 어떤 SWR-매핑에도 흡수되어 있지 않았기에 신규 도출되었다. 이 배정은 2026-09-18 사용자(가상 OEM-A 역할 겸임)가 확정했다(§8.2 결정 4). 더 이상 제안(미확정) 상태가 아니다.
- Upstream: OEM-IF-004 (사용자 확정 해석에 의해 신규 배정, OEM-FR-001에 준함) | Downstream: TBD
- 검증기준: side/action/source 각 필드에 대해 누락값·오형식·미등록 enum을 오류추정+결정테이블 기법으로 주입, 명령이 거절되고 좌·우 출력이 변경되지 않으며 오류 기록이 생성됨을 자동시험으로 검증(주입 케이스 100% 거절이 합격기준).
- 검토 상태: Draft

#### 4.1.2 OEM-FR-002 도출 — 자동 잠금 (SWR-003)

**SWR-003 — 차량 이동 시작 시 자동 잠금**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 유효한 vehicle_speed_kph가 3km/h 이상인 평가주기에는 후석 좌·우 도어 출력을 LOCK으로 설정해야 한다. 단, SWR-022 우선순위 정책상 이 트리거보다 상위 순위인 트리거(SWR-007/017/021/005/006/018/020)가 그 평가주기에 유효하면 그 트리거의 지시를 따른다.
- Upstream: OEM-FR-002, OEM-IF-001 | Downstream: TBD
- 검증기준: vehicle_speed_kph를 0→3.0km/h 이상으로 전이시키는 입력을 PC/SIL에 주입, 경계값(2.9/3.0/3.1km/h) 3점을 포함해 3.0km/h 이상이 되는 평가주기에 좌·우 출력이 LOCK임을 자동시험으로 검증. 아울러 vehicle_speed_kph≥3km/h 유지 중 운전자 unlock 명령(SWR-001/002)을 주입해 LOCK이 유지됨(SWR-022 순위7 > 순위8)을 검증.
- 검토 상태: Draft

#### 4.1.3 OEM-NFR-002 도출군 — 결정 레코드/순환보존 (SWR-010, 011)

**SWR-010 — 결정 레코드 스키마 정의**
- 유형: 기능(데이터 구조 요구, NFR 상위요구에서 도출) | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 매 평가주기의 제어결정을 기록할 때, 결정 레코드에 순번(sequence_id), 타임스탬프, lock_left, lock_right, state, reason_code 필드만 포함해야 하며, 영상·음성 원본 데이터 및 개인식별정보(성명, 얼굴, 음성 등)를 포함하지 않아야 한다.
- Upstream: OEM-NFR-002, 원문 2절 프로젝트 제약 | Downstream: TBD
- 검증기준: 결정 레코드 101건 생성 후 각 레코드의 필드셋이 허용 필드 목록과 완전히 일치하는지 스키마 검증 스크립트로 자동 검사(허용 외 필드 발견 0건이 합격기준).
- 검토 상태: Draft

**SWR-011 — 최근 100건 순환 보존(FIFO)**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 메모리 내 결정 저장소에 최근 100건의 결정 레코드만 유지하며, 101번째 레코드 생성 시 가장 오래된 레코드를 제거해야 한다.
- 근거: "1건"을 "평가주기당 1레코드"로 해석했다(§8.1 해석 3 — 비안전크리티컬, 재확인 권장 사항으로 §8.3에 등록되어 있으며 이번 개정의 4개 사용자 결정 대상은 아니다).
- Upstream: OEM-NFR-002 | Downstream: TBD
- 검증기준: 101건의 결정을 순차 생성한 뒤 저장소를 조회하여 최신 100건이 생성 순서대로 존재하고 가장 오래된 1건이 존재하지 않음을 단위시험으로 검증.
- 검토 상태: Draft

#### 4.1.4 OEM-FR-004 도출군 — 상태조회/표시 (SWR-014, 015)

**SWR-014 — 상태조회 응답 제공**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 상태조회 요청에 대해 현재 lock_left, lock_right, state(NORMAL/DEGRADED/FAULT/OFF), 각 안전관련 입력의 유효성(input_validity), 최근 결정의 reason_code를 응답으로 제공해야 한다.
- Upstream: OEM-FR-004 | Downstream: TBD
- 검증기준: 상태조회 함수/API를 호출해 응답 필드 집합이 요구된 5개 항목을 모두 포함하는지 통합시험으로 검증(필드 누락 0건이 합격기준).
- 검토 상태: Draft

**SWR-015 — 표시 인터페이스 데이터 제공(OEM-IF-006)**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 표시 인터페이스(OEM-IF-006)로 state, priority_reason, reason_code, input_validity를 제공해야 하며, priority_reason 값은 reason_code로도 동일하게 매핑되어야 한다. 직렬화에 실패하면 HTTP 500을 반환해야 한다.
- Upstream: OEM-FR-004, OEM-IF-006 | Downstream: TBD
- 검증기준: 각 트리거 시나리오(SWR-005~022 각 1케이스)에서 발생한 priority_reason이 reason_code 필드와 일치하는지 Web 통합시험으로 검증(불일치 0건이 합격기준), 직렬화 오류주입 시 HTTP 500 응답 여부 검증.
- 검토 상태: Draft

#### 4.1.5 OEM-FR-005 도출 — 화재/과열/성인탑승 강제해제 (SWR-017)

**SWR-017 — 화재/과열/성인탑승 강제해제**
- 유형: 기능 | ASIL: QM (OEM 지정 그대로 승계, 재도출하지 않음) | 우선순위(관리용): High
- 본문: SW는 fire_detected, overtemperature_detected, adult_present 중 하나 이상이 유효한 TRUE로 확인되면, crash_status=CONFIRMED(SWR-007)가 동시에 유효한 경우를 제외하고, 다음 평가주기에 후석 좌·우 도어 출력을 모두 RELEASE로 설정하고 트리거된 입력별 이유코드를 기록해야 한다. crash_status=CONFIRMED가 동시에 유효한 경우에도 결과 출력은 동일하게 RELEASE이며, 이유코드는 SWR-022 우선순위 정책에 따라 순위 1(SWR-007)의 근거로 기록된다.
- 근거: SWR-022 우선순위 정책(순위 2)에 따라 접근위험(SWR-005, 순위4)과 ISOFIX(SWR-018, 순위5)보다 우선하며, 센서고장(SWR-021, 순위3)이 동시에 유효해도 이 강제 RELEASE가 우선한다. 이는 2026-09-18 사용자 확정 결정 1의 직접 결과이다(§8.2).
- Upstream: OEM-FR-005, OEM-IF-007 | Downstream: TBD
- 검증기준: 3개 입력을 각각 단독 TRUE 주입(3케이스)하고, PICT 조합기법으로 2개/3개 동시 TRUE 조합(추가 4케이스)을 주입, 다음 평가주기에 두 출력이 RELEASE이고 입력별 이유코드가 기록됨을 자동시험으로 검증. 추가로 (fire_detected=TRUE, isofix_left=TRUE) 및 (fire_detected=TRUE, sensor_fault=TRUE, isofix_left=TRUE) 조합에서도 RELEASE가 유지됨(SWR-022 순위2 우선)을 검증.
- 검토 상태: Draft

#### 4.1.6 OEM-FR-006 도출 — ISOFIX 강제잠금 (SWR-018)

**SWR-018 — 좌/우 ISOFIX 강제잠금**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 isofix_left 또는 isofix_right가 TRUE로 확인되면, crash_status=CONFIRMED(SWR-007), fire_detected/overtemperature_detected/adult_present 중 TRUE(SWR-017), 또는 sensor_fault=TRUE(SWR-021)가 그 평가주기에 유효한 경우를 제외하고, 다음 평가주기에 해당 side의 도어 출력만 LOCK으로 설정하고 반대 side 출력은 변경하지 않아야 한다. ignition_on=FALSE(SWR-020)와 동시에 유효한 경우에는 이 ISOFIX 강제잠금이 우선하여 해당 side는 LOCK으로 유지된다.
- 근거: SWR-022 우선순위 정책(순위 5)에 따라 순위 1~3(충돌/화재등/센서고장)에는 종속되고, 순위 6(ignition-off) 및 순위 7~8(자동잠금/운전자명령)보다는 우선한다. 이는 2026-09-18 사용자 확정 결정 1의 직접 결과이다(§8.2).
- Upstream: OEM-FR-006, OEM-IF-008 | Downstream: TBD
- 검증기준: isofix_left만 TRUE, isofix_right만 TRUE, 둘 다 TRUE인 3케이스를 결정테이블 기법으로 주입, 해당 side LOCK·반대측 유지를 자동시험으로 검증. 추가로 (isofix_left=TRUE, ignition_on=FALSE) 조합에서 좌측이 LOCK으로 유지됨(SWR-022 순위5 우선)과, (isofix_left=TRUE, crash_status=CONFIRMED) 조합에서 RELEASE가 적용됨(순위1 우선)을 검증.
- 검토 상태: Draft

#### 4.1.7 OEM-FR-007 도출 — ignition-off (SWR-020)

**SWR-020 — ignition_on=FALSE 강제해제**
- 유형: 기능 | ASIL: QM | 우선순위(관리용): Medium
- 본문: SW는 ignition_on이 FALSE로 확인되는 평가주기에, SWR-022 우선순위 정책상 이보다 상위 순위인 트리거(SWR-007/017/021/005/006/018)가 유효하지 않은 도어에 대해 출력을 RELEASE로 전환하고 state를 OFF로, reason_code를 ignition_off로 설정해야 한다.
- 근거: SWR-022 우선순위 정책(순위 6)에 따라 ISOFIX(SWR-018, 순위5)보다 하위이며 자동잠금(SWR-003, 순위7)·운전자명령(순위8)보다는 상위이다. 이는 2026-09-18 사용자 확정 결정 1의 직접 결과이다(§8.2).
- Upstream: OEM-FR-007, OEM-IF-009 | Downstream: TBD
- 검증기준: ignition_on을 TRUE→FALSE로 전이시키는 입력을 주입, 전이 평가주기에 출력 RELEASE·state=OFF·reason_code=ignition_off임을 자동시험으로 검증. 추가로 (ignition_on=FALSE, isofix_left=TRUE) 조합에서 좌측은 LOCK(ISOFIX 우선), 우측은 RELEASE(ignition-off 적용)로 좌우가 다르게 결정됨을 검증(SWR-022 순위5>6).
- 검토 상태: Draft

### 4.2 안전 관련 SW 요구사항 (ASIL B)

> 안전 연계 공통 사항: Safety Goal → (OEM 소유, ISO 26262 Part 3 범위 밖) → OEM-SR-XXX(ASIL B 할당 입력) → 본 절의 SWR(Software Safety Requirement). FSR/TSR 문서는 OEM 소유이며 본 프로젝트 범위 밖이다.

#### 4.2.1 OEM-SR-001 도출군 — 충돌 우선 처리 (SWR-007, 008)

**SWR-007 — 충돌 CONFIRMED 강제해제**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 crash_status가 CONFIRMED로 확인되면, 운전자 명령(SWR-001/002)뿐 아니라 ISOFIX 강제잠금(SWR-018), 접근위험 LOCK 유지(SWR-005/006), 센서고장 유지(SWR-021)를 포함한 다른 모든 트리거보다 우선하여 후석 좌·우 도어 출력을 모두 RELEASE로 설정해야 하며, crash_status가 CONFIRMED로 유지되는 동안 이후 평가주기에서도 이 RELEASE를 유지해야 한다.
- 근거: OEM-SR-001 원문("다른 명령보다 우선")과 SWR-022 우선순위 정책(순위 1, 최우선)에 따름. ISOFIX/접근위험/센서고장과의 동시발생 우선순위는 2026-09-18 사용자 확정 결정 1로 해소되었다(§8.2, 이전 §0(b)-#2, #4).
- Upstream: OEM-SR-001, OEM-IF-002 | Downstream: TBD
- 검증기준: crash_status=CONFIRMED 주입 후 300ms 이내 좌·우 출력이 RELEASE인지 시간측정 자동시험으로 검증(300ms 초과 0건이 합격기준). 이후 5개 평가주기 연속 CONFIRMED 입력 시 RELEASE가 유지됨을 검증. 추가로 (crash_status=CONFIRMED, isofix_left=TRUE), (crash_status=CONFIRMED, rear_left_approach_risk=TRUE), (crash_status=CONFIRMED, sensor_fault=TRUE) 3개 조합에서도 RELEASE가 유지됨(순위1 최우선)을 PICT 조합시험으로 검증.
- ISO 26262 요구사항 특성 자체점검: 명확함(Pass — 상위 순위 및 하위 트리거와의 관계가 SWR-022로 명문화됨), 이해가능(Pass), 원자적(Pass), 내적일관성(Pass), 실현가능(Pass), 검증가능(Pass), 추상수준 적절(Pass), 추적가능(Pass, OEM-SR-001 ↔ Downstream TBD)
- 검토 상태: Draft

**SWR-008 — crash_status=PENDING 처리**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 crash_status가 PENDING인 평가주기에는 SWR-007의 충돌우선 강제해제를 발생시키지 않고, 해당 평가주기의 출력을 SWR-022 우선순위 정책에 따라 다른 적용 가능한 트리거(순위 2 이하)의 결과로 결정해야 한다. crash_status=PENDING 동안에도 다른 모든 트리거(화재/과열/성인탑승, 센서고장, 접근위험, ISOFIX, ignition-off, 자동잠금, 운전자 명령)는 SWR-022의 순위에 따라 정상적으로 평가된다.
- 근거: OEM-IF-002가 PENDING을 유효 열거값으로 정의하나, OEM-SR-001의 수용기준은 CONFIRMED에 대해서만 강제해제를 규정한다. PENDING에 대한 별도의 능동 동작은 정의하지 않는다는 것과, PENDING 동안 다른 트리거는 정상 평가된다는 것이 2026-09-18 사용자 확정 결정 3이다(§8.2, 이전 §0(b)-#8).
- Upstream: OEM-SR-001, OEM-IF-002 | Downstream: TBD
- 검증기준: crash_status=PENDING 입력 시 SWR-007의 강제 RELEASE가 트리거되지 않고, 동시 주입된 다른 정상 트리거(예: isofix_left=TRUE, 또는 운전자 lock 명령)가 SWR-022 순위에 따라 정상 반영됨을 PC/SIL 자동시험으로 검증.
- ISO 26262 요구사항 특성 자체점검: 명확함(Pass), 이해가능(Pass), 원자적(Pass), 내적일관성(Pass), 실현가능(Pass), 검증가능(Pass), 추상수준 적절(Pass), 추적가능(Pass)
- 검토 상태: Draft

#### 4.2.2 OEM-SR-002 도출군 — 접근위험 억제/재입력/좌우독립 (SWR-005, 006, 009)

**SWR-005 — 접근위험 도어 LOCK 및 해제 억제**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 rear_left_approach_risk 또는 rear_right_approach_risk가 TRUE인 도어의 출력을 LOCK으로 설정하고, 그 도어에 대한 unlock 명령을 적용하지 않고 원인(reason_code)을 기록해야 한다. 단, SWR-006의 override 조건이 성립하면 예외로 해당 도어 출력을 RELEASE로 전환하며, crash_status=CONFIRMED(SWR-007) 또는 fire_detected/overtemperature_detected/adult_present 중 TRUE(SWR-017)가 동시에 유효한 경우에는 SWR-022 우선순위 정책에 따라 그 강제 RELEASE 지시가 이 LOCK 지시보다 우선 적용된다.
- 근거: SWR-017과의 동시발생 우선순위(이전 §0(b)-#1)와 SWR-006 override의 실제 출력 전환 여부(이전 §0(b)-#5)는 2026-09-18 사용자 확정 결정 1, 2로 해소되었다(§8.2).
- Upstream: OEM-SR-002, OEM-IF-003 | Downstream: TBD
- 검증기준: (approach_risk TRUE/FALSE) × (해당 도어 unlock 명령 유무) 4케이스를 결정테이블로 주입, TRUE 시 LOCK 유지·reason_code 기록을 자동시험으로 검증. 추가로 (approach_risk=TRUE, fire_detected=TRUE) 조합에서 RELEASE가 적용됨(순위2 우선)과, override 성립 조합에서 RELEASE로 전환됨(SWR-006 참조)을 검증.
- ISO 26262 특성 자체점검: 명확함(Pass), 이해가능(Pass), 원자적(Pass), 내적일관성(Pass), 실현가능(Pass), 검증가능(Pass), 추상수준 적절(Pass), 추적가능(Pass, Upstream OEM-SR-002, Downstream TBD)
- 검토 상태: Draft

**SWR-006 — 접근위험 억제 후 10초 이내 재입력 override**
- 유형: 기능 | ASIL: **B** (SWR-005의 안전 메커니즘과 직결되어 QM으로 낮추지 않음) | 우선순위(관리용): High
- 본문: SW는 SWR-005에 의해 해제가 억제된 도어에 대해, 최초 억제 발생 시각으로부터 10초 이내에 동일 도어에 대한 동일 unlock 명령이 재입력되면, override 조건이 성립한 것으로 판단하여 해당 도어의 출력을 RELEASE로 전환하고, 결정 결과에 override 상태(override=TRUE) 및 override 이유코드를 함께 제공해야 한다. 경과시간이 10초를 초과하면 override 조건이 성립하지 않으며 SWR-005의 LOCK을 유지한다.
- 근거: 2026-09-18 사용자 확정 결정 2에 따라 override는 상태/이유코드 기록에 그치지 않고 실제로 도어 출력을 RELEASE로 전환하는 것으로 확정되었다(§8.2, 이전 §0(b)-#5). OEM-SR-002가 ASIL B로 할당되어 있고 override는 SWR-005 예외조건이므로, override 성립 시 RELEASE로 전환된 이후에도 crash_status=CONFIRMED 또는 SWR-017 트리거가 유효하면 그 결과는 이미 RELEASE이므로 SWR-022 순위와 충돌하지 않는다.
- Upstream: OEM-SR-002, OEM-FR-003 (양쪽 모두로부터 도출되는 다중 상위 요구사항) | Downstream: TBD
- 검증기준: approach_risk TRUE로 억제 발생 후 t=0s, 10.0s(경계), 10.1s에 동일 도어 unlock 재입력을 경계값분석으로 주입, t≤10s에서 override 상태/이유코드 생성과 함께 출력이 RELEASE로 전환됨을, t>10s에서 override 미생성 및 LOCK 유지됨을 검증(경계 3점 모두 기대값과 일치가 합격기준).
- ISO 26262 특성 자체점검: 명확함(Pass — 출력 전환 여부가 확정 서술됨), 이해가능(Pass), 원자적(Pass), 내적일관성(Pass), 실현가능(Pass), 검증가능(Pass), 추상수준 적절(Pass), 추적가능(Pass)
- 검토 상태: Draft

**SWR-009 — 좌우 접근위험 독립 평가**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 rear_left_approach_risk와 rear_right_approach_risk를 서로 독립적으로 평가하여, 한쪽만 TRUE인 경우 해당 side의 출력에만 SWR-005를 적용하고 반대 side 출력에는 영향을 주지 않아야 한다.
- Upstream: OEM-SR-002, OEM-IF-003 | Downstream: TBD
- 검증기준: (TRUE,FALSE), (FALSE,TRUE), (TRUE,TRUE), (FALSE,FALSE) 4가지 조합을 결정테이블로 주입, 각 side 출력이 독립적으로 결정됨을 자동시험으로 검증(4/4 통과가 합격기준).
- ISO 26262 특성 자체점검: 전항목 Pass.
- 검토 상태: Draft

#### 4.2.3 OEM-SR-003 도출군 — 입력 유효성/freshness (SWR-013-A, 013-B)

> 명확성 체크리스트(단문·원자적 요구) 준수를 위해 원문의 2개 조건(freshness→DEGRADED, 형식/범위→거절)을 SWR-013-A/013-B로 세분화했다. 상위 매핑은 OEM 원문이 지정한 단일 ID "SWR-013" 그룹으로 유지한다.

**SWR-013-A — 입력 freshness 감시 및 DEGRADED 전이**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 안전관련 입력(vehicle_speed_kph, gear, crash_status, rear_left_approach_risk, rear_right_approach_risk, fire_detected, overtemperature_detected, adult_present, isofix_left, isofix_right, ignition_on, sensor_fault — OEM-IF-001/002/003/007/008/009 전체)의 source_timestamp_s 기준 갱신 간격이 200ms를 초과하면, 그 사실을 감지한 시점부터 100ms 이내에 state를 DEGRADED로 전환해야 한다.
- 근거: "안전 관련 입력"의 정확한 필드 목록이 원문에 열거되지 않아, Vehicle→SW 인터페이스 전체로 범위를 설정했다(§8.1 해석 1 — 비안전크리티컬 방향의 보수적 해석이며, 재확인 권장 사항으로 §8.3에 등록되어 있고 이번 개정의 4개 사용자 결정 대상은 아니다).
- Upstream: OEM-SR-003, OEM-IF-001/002/003/007/008/009 | Downstream: TBD
- 검증기준: 특정 입력 채널의 갱신을 200ms 초과 정지시키고, 감지 후 100ms 이내 state=DEGRADED 전환 여부를 경계값분석(199ms/200ms/201ms)으로 자동시험 검증.
- ISO 26262 특성 자체점검: 명확함(Pass, 대상 필드 범위는 보수적으로 확정 — 재확인 권장은 §8.3 참조), 나머지 Pass.
- 검토 상태: Draft

**SWR-013-B — 형식/범위 오류 입력 거절**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 OEM-IF-001~IF-009에 정의된 형식/범위를 벗어나거나 누락된 입력 필드를 갖는 평가주기에 대해, 그 값을 결정 로직 평가에 사용하기 전에 거절(INVALID 처리)해야 한다.
- Upstream: OEM-SR-003, OEM-IF-001~009 | Downstream: TBD
- 검증기준: 각 인터페이스별 경계값/오류값(예: vehicle_speed_kph = -1, 300.1; gear = "X"; crash_status = "UNKNOWN")을 동등분할+경계값분석 기법으로 주입, 평가 로직에 반영되지 않고 INVALID로 거절됨을 자동시험으로 검증(전 케이스 100% 거절이 합격기준).
- ISO 26262 특성 자체점검: 전항목 Pass.
- 검토 상태: Draft

#### 4.2.4 OEM-SR-004 도출군 — 센서고장 유지 (SWR-021)

**SWR-021 — sensor_fault 게이트 및 출력 유지 (순위 3, 충돌·화재등 예외)**
- 유형: 기능 | ASIL: **B** | 우선순위(관리용): High
- 본문: SW는 정규화된 입력의 sensor_fault가 TRUE로 확인되는 평가주기에는, crash_status=CONFIRMED(SWR-007) 또는 fire_detected/overtemperature_detected/adult_present 중 TRUE(SWR-017)가 그 평가주기에 함께 유효한 경우를 **제외하고**, 접근위험(SWR-005/006/009)·ISOFIX(SWR-018)·ignition-off(SWR-020)·자동잠금(SWR-003)·운전자 명령(SWR-001/002/004/019)을 포함한 다른 모든 트리거의 평가 결과를 적용하지 않고, 직전 확정된 lock_left, lock_right 출력을 그대로 유지하며, state를 FAULT로, 정의된 경고코드를 함께 제공해야 한다. crash_status=CONFIRMED 또는 SWR-017 트리거가 동시에 유효한 경우에는 그 트리거의 강제 RELEASE 지시가 이 출력유지 지시보다 우선 적용된다.
- 근거: OEM-SR-004 "새 명령을 적용하지 않고 직전 확정 출력을 유지해야 한다"의 "새 명령"을 이 프로젝트에서는 모든 트리거의 해당 평가주기 결과로 넓게 해석한 것은 이전 개정(Rev 0.1)과 동일하다. 다만 sensor_fault가 crash CONFIRMED·화재/과열/성인탑승보다도 우선하는지는 이전에 미확정이었으며(이전 §0(c)), 2026-09-18 사용자 확정 결정 1에 따라 **sensor_fault는 SWR-022 우선순위 정책의 순위 3으로 확정**되어 순위 1(SWR-007)·순위 2(SWR-017)보다 하위임이 명확해졌다(§8.2). 이는 Rev 0.1의 "센서고장이 crash CONFIRMED를 무력화한다"는 결론을 **정정**하는 중요한 변경이다.
- Upstream: OEM-SR-004, OEM-IF-009 | Downstream: TBD
- 검증기준: sensor_fault=TRUE와 (crash_status=CONFIRMED, fire_detected=TRUE, isofix_left=TRUE, 운전자 unlock 명령) 각각/조합을 PICT 조합시험으로 주입: (a) crash_status=CONFIRMED 또는 fire_detected=TRUE와 동시 발생 시 출력이 RELEASE(순위1/2 우선)임을, (b) 그 외 모든 조합(ISOFIX/ignition-off/자동잠금/운전자명령과의 동시발생)에서는 출력이 직전 확정값으로 유지되고 state=FAULT·경고코드가 제공됨을 검증(전 조합 100% 기대값 일치가 합격기준).
- ISO 26262 특성 자체점검: 명확함(Pass — 예외조건이 명문화됨), 이해가능(Pass), 원자적(Pass), 내적일관성(Pass), 실현가능(Pass), 검증가능(Pass), 추상수준 적절(Pass), 추적가능(Pass)
- 검토 상태: Draft (**중요**: Rev 0.1 대비 동작이 정정되었으므로 검토 시 특히 유의 — 하위 아키텍처/구현 산출물이 아직 없어 영향 범위는 본 SWE.1 단계로 한정됨)

#### 4.2.5 신규 — 트리거 간 우선순위 결정 정책 (SWR-022)

**SWR-022 — 트리거 간 8단계 우선순위 결정 정책**
- 유형: 기능(우선순위 결정 정책) | ASIL: **B** | 우선순위(관리용): High
- 안전 연계: Safety Goal → (OEM 소유, Part 3 범위 밖) → OEM-SR-001, OEM-SR-002(양쪽 모두 ASIL B 할당 입력) → 본 SWR. FSR/TSR은 OEM 소유(범위 밖).
- 본문: SW는 매 평가주기마다 아래 표의 순위 1부터 순위 8까지의 순서로 각 트리거 조건의 유효성을 판정해야 하며, 순위가 더 높은(표의 번호가 더 작은) 트리거가 그 평가주기에 유효한 동안에는, 순위가 더 낮은(번호가 더 큰) 트리거가 지시하는 출력을 적용하지 않아야 한다. 동일 순위 내부의 예외 규칙(순위 4의 override 예외, SWR-006)은 해당 SWR에 정의된 대로 그 순위 내에서 적용된다.

  | 순위 | 트리거 조건 | 지시 | 관련 SWR |
  |---|---|---|---|
  | 1 | crash_status = CONFIRMED | 좌·우 강제 RELEASE | SWR-007 |
  | 2 | fire_detected 또는 overtemperature_detected 또는 adult_present 중 TRUE | 좌·우 강제 RELEASE | SWR-017 |
  | 3 | sensor_fault = TRUE (단, 순위 1·2가 그 평가주기에 함께 유효하지 않은 경우에만 적용) | 직전 확정 출력 유지, state=FAULT | SWR-021 |
  | 4 | rear_left_approach_risk 또는 rear_right_approach_risk = TRUE | 해당 도어 LOCK + 해제 억제(단, SWR-006 override 조건 성립 시 예외로 해당 도어 RELEASE) | SWR-005, SWR-006 |
  | 5 | isofix_left 또는 isofix_right = TRUE | 해당 side 강제 LOCK | SWR-018 |
  | 6 | ignition_on = FALSE | 좌·우 강제 RELEASE + state=OFF | SWR-020 |
  | 7 | vehicle_speed_kph ≥ 3km/h | 좌·우 LOCK | SWR-003 |
  | 8 | 운전자 명령(physical_button/avn/voice/mobile_app, side/action 지정) | 지정된 side/action에 따른 LOCK/UNLOCK | SWR-001, SWR-002, SWR-004, SWR-019 |

- 근거: **OEM 원문(OEM-SWR-001)에는 8개 트리거 전체를 아우르는 순위가 명시되어 있지 않다.** OEM-SR-001은 충돌 CONFIRMED가 "다른 명령보다 우선"함만 규정하고, OEM-SR-002는 접근위험이 해제 요청을 억제함만 규정하며, 두 요구 모두 서로 간 또는 다른 자동 트리거(화재/과열/성인탑승, ISOFIX, 센서고장, 자동잠금, ignition-off)와의 관계는 특정하지 않는다. 위 8단계 순서는 **원문에 존재하는 근거가 아니라, 이 프로젝트가 사용자(가상 OEM-A 역할을 겸임하는 jay.kim3063@gmail.com)의 명시적 승인을 받아 2026-09-18에 정의한 정책 요구사항**이다(§8.2 결정 1). 실제 근거가 있는 것처럼 위장하지 않기 위해, 이 사실을 이 SWR의 근거 필드에 명확히 기록한다. 두 개의 명시적 우선순위 규정(SWR-007 유래 OEM-SR-001, SWR-005 유래 OEM-SR-002)을 모두 포괄하는 정책이므로 Upstream을 양쪽 모두로 잡는다.
- Upstream: OEM-SR-001, OEM-SR-002 (원문에 전체 순서 근거 없음 — 사용자 승인으로 신규 정의, 위 근거 참조) | Downstream: TBD
- 검증기준: 결정테이블/PICT 조합기법으로 인접 순위쌍 7개(1-2, 2-3, 3-4, 4-5, 5-6, 6-7, 7-8) 각각을 동시 활성화한 케이스와, 비인접 대표쌍(1-5, 1-8, 2-8, 3-1&2 동시, 1-2-3 동시)을 포함해 최소 12개 조합을 자동시험으로 주입, 매 케이스에서 실제 출력이 "그 케이스에서 유효한 트리거 중 가장 높은 순위(가장 작은 번호)가 지시하는 값"과 일치하는지 검증(불일치 0건이 합격기준).
- ISO 26262 요구사항 특성 자체점검: 명확함(Pass — "적용되지 않는다"는 검증 가능한 문장으로 서술됨), 이해가능(Pass), 원자적(부분 — 8개 순위를 한 표로 규정하는 정책성 요구이므로 원자성은 "하나의 정책"으로서 판단, 개별 순위쌍 검증은 위 검증기준에서 분리됨), 내적일관성(Pass — 각 순위값은 상호 배타적 서수), 실현가능(Pass), 검증가능(Pass — 조합시험으로 100% 판정 가능), 추상수준 적절(Pass), 추적가능(Pass, Upstream 이중 명시, Downstream TBD)
- 검토 상태: Draft (신규, 검토 대기)

---

## 5. 입력 데이터 사전

| 신호명 | 자료형 | 단위 | 범위/유효값 | 유효성 조건(freshness) | 오류 처리 |
|---|---|---|---|---|---|
| vehicle_speed_kph | 실수 | km/h | 0.0 이상(상한은 원문에 상세 열거되지 않음 — SWR-013-B 시험케이스에서 -1, 300.1을 오류값으로 확인) | source_timestamp_s 기준 200ms 이내 갱신(SWR-013-A) | 범위/형식 오류 시 SWR-013-B에 따라 INVALID 거절 |
| gear | 열거형 | — | OEM-IF-001에 열거값이 상세 정의되지 않음(TBD, 원문 미상세) | 200ms 이내 갱신(SWR-013-A) | 형식오류 시 SWR-013-B에 따라 INVALID 거절 |
| source_timestamp_s | 실수 | 초 | 단조 증가 가정(원문 미상세) | 전 인터페이스 공통 freshness 판단 기준(SWR-013-A) | 갱신 정지 시 200ms 초과로 DEGRADED |
| crash_status | 열거형 | — | {NONE, PENDING, CONFIRMED} | 200ms 이내 갱신(SWR-013-A) | 미등록 값 시 SWR-013-B에 따라 INVALID 거절 |
| rear_left_approach_risk / rear_right_approach_risk | 불리언 | — | {TRUE, FALSE} | 200ms 이내 갱신(SWR-013-A) | 형식오류 시 SWR-013-B에 따라 INVALID 거절 |
| side / action / source (운전자 명령) | 열거형 | — | side∈{left,right,all}; action∈{lock,unlock}; source∈{physical_button,avn,voice,mobile_app} | 명령 채널 고유(freshness 대상 아님, OEM-IF-004는 §4.2.3의 안전관련 입력 목록에 포함되지 않음) | 누락/오형식/미등록 enum 시 SWR-019에 따라 거절 |
| lock_left / lock_right (출력) | 열거형 | — | {LOCK, RELEASE} | 해당 없음(SW 출력) | 해당 없음 |
| state / priority_reason / reason_code / input_validity (표시 출력) | 복합 | — | state∈{NORMAL, DEGRADED, FAULT, OFF} | 해당 없음(SW 출력) | 직렬화 실패 시 SWR-015에 따라 HTTP 500 |
| fire_detected / overtemperature_detected / adult_present | 불리언 | — | {TRUE, FALSE} | 200ms 이내 갱신(SWR-013-A) | 형식오류 시 SWR-013-B에 따라 INVALID 거절 |
| isofix_left / isofix_right | 불리언 | — | {TRUE, FALSE} | 200ms 이내 갱신(SWR-013-A) | 형식오류 시 SWR-013-B에 따라 INVALID 거절 |
| ignition_on / sensor_fault | 불리언 | — | {TRUE, FALSE} | 200ms 이내 갱신(SWR-013-A) | 형식오류 시 SWR-013-B에 따라 INVALID 거절 |

비고: "gear" 및 vehicle_speed_kph 상한 등 원문에 상세 열거되지 않은 값은 TBD로 명시했으며, 임의로 구체값을 추정하지 않았다(§8.3 참조 대상은 아니며, 단순 정보 부족으로 기록).

---

## 6. 외부 인터페이스 요구

| 인터페이스 ID | 방향 | 주요 필드 | 사용 SWR | 오류 처리 SWR |
|---|---|---|---|---|
| OEM-IF-001 | Vehicle → SW | vehicle_speed_kph, gear, source_timestamp_s | SWR-003, SWR-013-A/B | SWR-013-A(freshness), SWR-013-B(형식/범위) |
| OEM-IF-002 | Vehicle → SW | crash_status | SWR-007, SWR-008, SWR-022 | SWR-013-A/B |
| OEM-IF-003 | Vehicle → SW | rear_left/right_approach_risk | SWR-005, SWR-006, SWR-009, SWR-022 | SWR-013-A/B |
| OEM-IF-004 | Driver/HMI → SW | side, action, source | SWR-001, SWR-002, SWR-004, SWR-019, SWR-022 | SWR-019 |
| OEM-IF-005 | SW → Actuator | lock_left, lock_right | 전 출력결정 SWR 공통 | 해당 없음 |
| OEM-IF-006 | SW → Display | state, priority_reason, reason_code, input_validity | SWR-014, SWR-015 | SWR-015(직렬화 실패 시 HTTP 500) |
| OEM-IF-007 | Vehicle → SW | fire_detected, overtemperature_detected, adult_present | SWR-017, SWR-022 | SWR-013-A/B |
| OEM-IF-008 | Vehicle → SW | isofix_left, isofix_right | SWR-018, SWR-022 | SWR-013-A/B |
| OEM-IF-009 | Vehicle → SW | ignition_on, sensor_fault | SWR-020, SWR-021, SWR-022 | SWR-013-A/B |

---

## 7. 비기능 및 환경 제약

| SWR | 품질특성(ISO 25010) | 정량적 기준 | 검증방안(방법+합격기준) |
|---|---|---|---|
| SWR-012 | 보안성(기밀성) | 재기동 후 레코드 0건, PII 필드 0건 | 통합시험(재기동 후 조회) + 정적분석(필드명 검사), 각 0건이 합격기준 |
| SWR-016 | 신뢰성(성숙성), 부수적으로 기능 적합성(정확성) | 고정 벡터 1,000회 재생 시 해시 100% 동일 | 자동 회귀시험 스크립트(SHA-256 해시 비교, Python 3.12/unittest), 불일치 0건이 합격기준 |

**SWR-012 — 저장소 휘발성 및 PII 미저장** (비기능): SW는 결정 저장소를 프로세스 메모리 내에만 유지해야 하며, 프로세스가 재시작되면 이전 결정 레코드가 전혀 남아있지 않아야 한다. Upstream: OEM-NFR-002, 원문 2절. Downstream: TBD. 검토 상태: Draft.

**SWR-016 — 고정 입력 벡터 결정론적 재현성** (비기능): SW는 동일한 초기 상태와 동일한 순서의 입력 벡터가 고정된 시계로 재생될 때 매 회 동일한 제어결과(공개 결과 필드 기준)를 산출해야 한다. Upstream: OEM-NFR-001. Downstream: TBD. 검토 상태: Draft.

**환경 제약**: 참조 구현/자동시험 환경은 Python 3.12(§1.3). 검증 범위는 PC/SIL 및 Web(§1.3)로 한정된다. 영상·음성 원본·PII는 입력·저장하지 않는다(§1.3, SWR-010/012).

---

## 8. 분석 결과와 가정

### 8.1 해석 및 가정 목록

원문만으로 결정할 수 없었던 항목 중, 임의로 보류하지 않고 논리적으로 도출해 명시적으로 반영한 해석은 다음과 같다.

1. **입력 freshness 대상 범위(SWR-013-A)**: OEM-SR-003의 "안전 관련 입력"이 정확히 어떤 필드 집합을 가리키는지 원문이 열거하지 않아, Vehicle→SW 인터페이스 전체(OEM-IF-001, 002, 003, 007, 008, 009)로 보수적으로(더 넓게) 해석했다. 과소포함보다 과대포함이 안전 측면에서 더 보수적이므로 안전크리티컬 재확인 필요 사항으로 분류하지 않으며, §8.3에 재확인 권장으로만 등록한다.
2. **NFR-002 "1건"의 단위(SWR-010/011)**: OEM-NFR-002 "최근 제어결정 100건"의 1건이 "매 평가주기마다 1건"인지 "출력이 변경될 때만 1건"인지 원문이 명시하지 않는다. 수용기준의 "101건 입력 후"라는 표현을 근거로 매 평가주기 = 1레코드로 해석했다. §8.3에 재확인 권장으로 등록한다.
3. (참고, 상동) 위 2건은 이번 개정(Rev 0.2)에서 사용자에게 확인을 요청한 4개 결정 항목에 포함되지 않았으며, 안전영향이 낮아 §8.3의 잔존 사항으로 이월한다.

### 8.2 해결된 사용자 확인 이력 (감사기록)

이 절은 Rev 0.1에서 "사용자 확인 필요"로 남겨졌던 항목 중, 2026-09-18 사용자(가상 OEM-A 역할을 겸임하는 jay.kim3063@gmail.com)의 결정으로 해소된 4건을 무엇을 왜 그렇게 결정했는지와 함께 기록한다. 감사 추적 목적상 삭제하지 않고 보존한다.

**결정 1 — 트리거 간 전체 우선순위 정책 (신규 SWR-022)**
- 이전 미결 상태: Rev 0.1 §0(b)의 8개 충돌 쌍(#1~#8 중 순위 관련 #1,#2,#3,#4,#6,#7)과 §0(c)의 sensor_fault 최상위 게이트 해석(안전영향 매우 높음, crash CONFIRMED 무력화 우려).
- 결정: 아래 8단계 순위를 확정한다(1이 최우선): ① crash CONFIRMED 강제 RELEASE, ② 화재/과열/성인탑승 강제 RELEASE, ③ 센서고장 마지막 출력 유지(①·②가 동시에 유효하지 않을 때만), ④ 접근위험 LOCK+억제(단 override 예외), ⑤ ISOFIX 강제 LOCK, ⑥ ignition-off 강제 RELEASE, ⑦ 자동잠금(속도≥3km/h) LOCK, ⑧ 운전자 명령.
- 근거/영향: OEM 원문에는 이 전체 순서가 명시되어 있지 않다(원문은 SWR-007과 SWR-005 두 개의 개별 우선순위만 규정). 이 정책은 원문 근거가 아니라 **사용자 승인에 의해 정의된 정책 요구사항**이며, SWR-022로 명문화했다(§4.2.5). 이 결정의 가장 중요한 결과는 Rev 0.1의 SWR-021(sensor_fault 최상위 게이트) 해석을 **정정**한 것이다 — sensor_fault는 이제 순위 3으로, crash CONFIRMED·화재등보다 하위이다.
- 반영 SWR: SWR-003, 004, 005, 006, 007, 008(간접), 017, 018, 020, 021, 022(신규).
- 결정일 / 결정자: 2026-09-18 / jay.kim3063@gmail.com(가상 OEM-A 역할 겸임).

**결정 2 — SWR-006 Override의 실제 출력 전환**
- 이전 미결 상태: Rev 0.1 §0(b)-#5. override가 상태/이유코드만 기록하는지, 실제 RELEASE로 전환하는지 미정.
- 결정: override 성립 시 실제로 해당 도어 출력을 RELEASE로 전환하고, override 상태·이유코드를 함께 제공한다.
- 근거: OEM-FR-003의 취지("명시적 override로 처리")가 상태 기록에 그치지 않고 실질적 해제 허용을 의도한다는 사용자 판단.
- 반영 SWR: SWR-005(예외조항 추가), SWR-006(본문 확정).
- 상태 정정: SWR-005, SWR-006을 "Draft(부분)"에서 "Draft"(일반)로 정정.
- 결정일 / 결정자: 2026-09-18 / jay.kim3063@gmail.com.

**결정 3 — crash_status=PENDING 처리 범위**
- 이전 미결 상태: Rev 0.1 §0(b)-#8. PENDING 동안 능동적으로 수행해야 할 동작(경고 표시 등)이 원문에 규정되지 않음.
- 결정: 별도의 명시적 능동 동작을 정의하지 않는다. 기존 SWR-008("PENDING은 SWR-007의 강제해제를 유발하지 않는다")을 유지하고, "PENDING 동안 다른 모든 트리거는 SWR-022 우선순위 정책에 따라 정상 평가된다"는 문장을 추가해 명확화한다.
- 반영 SWR: SWR-008.
- 결정일 / 결정자: 2026-09-18 / jay.kim3063@gmail.com.

**결정 4 — SWR-019 확정**
- 이전 미결 상태: Rev 0.1 §0(a). SWR-019의 내용 배정(운전자 명령 필드 유효성 검증/거절)이 "제안(미확정)"으로 표기됨.
- 결정: SWR-019 = "운전자 명령 필드(OEM-IF-004: side/action/source) 유효성 검증 및 미등록 enum 거절"로 확정한다.
- 반영 SWR: SWR-019. 상태 정정: "Draft(§0(a) 확인 대기)"에서 "Draft"(일반)로 정정.
- 결정일 / 결정자: 2026-09-18 / jay.kim3063@gmail.com.

### 8.3 잔존 사항 (비안전크리티컬, 해석적)

아래 항목은 이번 개정의 4개 사용자 결정 대상에 포함되지 않았으며, 안전크리티컬하지 않은 해석적 사항으로 남긴다. 임의로 결정을 위장하지 않고 재확인을 권장하는 상태임을 명시한다.

- **(잔존-1)** §8.1-1: SWR-013-A의 "안전 관련 입력" 범위(Vehicle→SW 인터페이스 전체로 해석)에 대한 재확인 권장.
- **(잔존-2)** §8.1-2: SWR-010/011의 "1건 = 평가주기당 1레코드" 해석에 대한 재확인 권장.
- **(잔존-3, 이번 개정에서 신규 식별)** SWR-022의 8단계 순위는 LOCK/RELEASE/유지라는 **출력 우선순위**만 규정한다. state 필드 값 중 DEGRADED(SWR-013-A, freshness 위반)는 이 8단계 표에 포함되어 있지 않아, DEGRADED와 나머지 7개 트리거(특히 FAULT/OFF)가 동시에 성립할 때 어떤 state 값이 표시되어야 하는지는 이번 개정에서도 명확히 정의되지 않았다. 안전에 직접 영향(도어 LOCK/RELEASE 결정)을 주는 사항은 아니며 표시(SWR-014/015)에만 관련되므로 비안전크리티컬로 분류하나, 다음 개정에서 사용자 확인을 권장한다.

---

## 9. 하향 할당 및 검증 계획

| SWR | 검증수준 | 검증기준 요약(합격기준) | 설계 할당 대상(Downstream) |
|---|---|---|---|
| SWR-001 | PC/SIL | 16조합 100% 기대값 일치 | TBD |
| SWR-002 | PC/SIL | 8조합 100% 기대값 일치 | TBD |
| SWR-003 | PC/SIL | 경계값 3점 + 순위7>8 검증 | TBD |
| SWR-004 | PC/SIL | 결정테이블 7조건×반대명령, override 예외 별도 | TBD |
| SWR-005 | PC/SIL | 결정테이블 4케이스 + 순위2/override 조합 | TBD |
| SWR-006 | PC/SIL | 경계값 3점(0/10.0/10.1s) 전환 검증 | TBD |
| SWR-007 | PC/SIL | 300ms 시간측정 + 3개 조합(순위1 최우선) | TBD |
| SWR-008 | PC/SIL | PENDING 시 비강제 + 타 트리거 정상평가 | TBD |
| SWR-009 | PC/SIL | 4조합 100% | TBD |
| SWR-010 | 단위 | 스키마 검증, 허용외 필드 0건 | TBD |
| SWR-011 | 단위 | FIFO 101건 검증 | TBD |
| SWR-012 | 통합 | 재기동 후 0건 + 정적분석 0건 | TBD |
| SWR-013-A | PC/SIL | 경계값 199/200/201ms | TBD |
| SWR-013-B | 단위/PC/SIL | 동등분할+경계값 100% 거절 | TBD |
| SWR-014 | 통합 | 응답필드 5종 누락 0건 | TBD |
| SWR-015 | Web 통합 | priority_reason=reason_code 불일치 0건, 500 검증 | TBD |
| SWR-016 | PC/SIL | 해시 1,000회 100% 동일 | TBD |
| SWR-017 | PC/SIL/Web | 7케이스 + 순위2 우선 조합 | TBD |
| SWR-018 | PC/SIL/Web | 3케이스 + 순위5 상호작용 조합 | TBD |
| SWR-019 | PC/SIL | 오류주입 100% 거절 | TBD |
| SWR-020 | PC/SIL/Web | 전이검출 + 순위5>6 조합 | TBD |
| SWR-021 | PC/SIL | PICT 조합, 순위1/2 예외 포함 100% | TBD |
| SWR-022 | PC/SIL | 인접쌍 7 + 비인접 대표쌍 최소 12조합 100% | TBD |

Downstream(아키텍처/상세설계/코드/SWE.4~6)은 아직 존재하지 않으므로 전량 TBD이다. 상세 양방향 추적은 `TRC-001_양방향 요구사항 추적 매트릭스.md` 참조(§11).

---

## 10. 범위 밖 주장

- 본 문서와 하위 산출물만으로는 ISO 26262 인증, Automotive SPICE 공식 심사, HARA/ASIL 결정의 타당성을 주장할 수 없다.
- HIL 시험, 실차 시험, 타깃 ECU 통합, 하드웨어 개발, Part 3(HARA) 활동은 본 프로젝트 범위 밖이며 본 문서의 어떤 검증기준도 이를 대체하지 않는다.
- ASIL B "할당"은 OEM 입력으로 그대로 수용했으며, 그 할당의 타당성 자체(HARA 결과 재확인)는 본 문서의 범위가 아니다.

---

## 11. 추적성

상세 양방향 추적 매트릭스는 `WorkProducts/Engineering/Traceability/TRC-001_양방향 요구사항 추적 매트릭스.md`에 있다(TPL-TRC-001 열 구성: Upper Req/SW Req/Architecture/Detailed Design/Code/SWE.4/SWE.5/SWE.6/Coverage). 요약:

- OEM 요구사항 13건 → SWR 22건(SWR-013은 013-A/013-B로 세분화, SWR-022 신규 추가로 실질 23개 레코드) 전량 도출 완료.
- SWR-019는 OEM 추적성 표의 빈 칸을 신규 해석으로 채운 뒤 사용자 확정을 받았다(§8.2 결정 4).
- SWR-006은 OEM-SR-002와 OEM-FR-003 양쪽에서 도출되는 다중 Upstream 레코드이다.
- SWR-022는 OEM-SR-001과 OEM-SR-002 양쪽에서 도출되는 다중 Upstream 레코드이며, 원문에 전체 순서 근거가 없다는 점이 §4.2.5/§8.2에 명시되어 있다.
- 모든 SWR의 Downstream(Architecture~Coverage)은 아키텍처/상세설계/코드/테스트 산출물이 아직 없으므로 전량 TBD이다(설계 착수 전 정상 상태).
- Use Case 문서(`SWE1-002`)와의 관계는 해당 문서 §6(추적성)에 정리되어 있다.

---

## 12. 참고자료

- 적용 표준(실무 보조 참고, 원문 대체 아님): ISO 26262(도로차량 기능안전), Automotive SPICE(A-SPICE) 4.1 PAM.
- 고객 입력 식별정보: `OEM_Sample/OEM-SWR-001_OEM SW 요구사항 사양서.docx`, Rev 1.0, BL-OEM-1.0(가상 OEM-A, 교육용).
- 고객 요구사항 원문 인용(참고용, System 레벨, OEM-SWR-001 원문 5절 그대로 인용 — 재해석·수정 금지, 레벨: System/OEM 입력):

| ID | 분류 | 요구사항(원문) | 수용기준(원문) | 검증수준(원문) | 하위 SWR |
|---|---|---|---|---|---|
| OEM-SR-001 | ASIL B 입력 | SW는 OEM 충돌상태가 CONFIRMED가 되면 다른 명령보다 우선하여 후석 좌, 우 잠금 해제를 요구해야 한다. | 유효 충돌 입력 시 두 출력이 300ms 이내 RELEASE이고 이후 입력 주기에도 유지된다. | PC/SIL SW 검증 | SWR-007, SWR-008 |
| OEM-SR-002 | ASIL B 입력 | SW는 후측방 접근위험이 TRUE인 도어를 LOCK하고 해당 도어의 해제 요청을 억제해야 한다. | 접근위험 TRUE가 되면 해당 출력이 LOCK이고 RELEASE 요청에도 LOCK을 유지하며 원인이 기록된다. | PC/SIL SW 검증 | SWR-005, SWR-006, SWR-009 |
| OEM-SR-003 | ASIL B 입력 | SW는 안전 관련 입력의 유효성과 freshness를 확인하고 비정상 입력을 정의된 방식으로 처리해야 한다. | 필수 입력이 200ms를 초과해 갱신되지 않으면 100ms 이내 DEGRADED 상태가 되고, 형식/범위 오류는 평가 전에 거절된다. | 단위 및 PC/SIL | SWR-013-A, SWR-013-B |
| OEM-SR-004 | ASIL B 입력 | SW는 정규화 입력의 sensor_fault가 TRUE이면 새 명령을 적용하지 않고 직전 확정 출력을 유지해야 한다. | sensor_fault=TRUE인 첫 평가주기에 좌, 우 출력이 유지되고 FAULT 상태와 경고코드가 제공된다. | 단위 및 PC/SIL | SWR-021 |
| OEM-FR-001 | QM | SW는 물리 버튼, AVN, 음성 및 모바일 앱 경로로 수신한 운전자의 좌, 우, 전체 잠금 및 해제 명령을 처리해야 한다. | 정지, 정상입력에서 네 source 각각의 LOCK/RELEASE와 LEFT/RIGHT/ALL 조합이 선택 출력에만 적용된다. | PC/SIL/Web | SWR-001, SWR-002, SWR-004 |
| OEM-FR-002 | QM | SW는 차량이 이동을 시작하면 후석 좌, 우 차일드락을 자동으로 활성화해야 한다. | 유효 차속이 3km/h 이상인 첫 평가주기부터 두 출력이 LOCK이다. | PC/SIL | SWR-003 |
| OEM-FR-003 | QM | SW는 운전자가 접근위험 경고 후 10초 이하에 같은 해제 명령을 다시 입력하면 명시적 override로 처리해야 한다. | 첫 억제 후 경과시간이 10초 이하인 같은 도어 재입력에서 override 상태와 이유코드가 생성된다. | PC/SIL/Web | SWR-006 |
| OEM-FR-004 | QM | SW는 현재 잠금, 상태, 입력 유효성 및 최근 결정 이유를 표시 인터페이스에 제공해야 한다. | 상태조회 응답에 좌, 우 출력, state, 입력유효성 및 이유코드가 포함된다. | 통합 및 Web | SWR-014, SWR-015 |
| OEM-FR-005 | QM | SW는 화재, 과온 또는 성인 탑승이 유효한 불리언 입력으로 확인되면 후석 좌, 우 차일드락을 강제로 해제해야 한다. | 각 입력을 단독 주입했을 때 다음 평가주기에 두 출력이 RELEASE이고 입력별 이유코드가 기록된다. | PC/SIL/Web | SWR-017 |
| OEM-FR-006 | QM | SW는 좌, 우 ISOFIX 연결 입력에 따라 해당 후석 차일드락을 강제로 활성화해야 한다. | 한쪽 ISOFIX만 TRUE이면 다음 평가주기에 해당 출력만 LOCK이고 반대쪽 출력은 유지된다. | PC/SIL/Web | SWR-018 |
| OEM-FR-007 | QM | SW는 ignition_on이 FALSE이면 논리 차일드락 출력을 초기 해제 상태로 전환해야 한다. | ignition_on=FALSE인 첫 평가주기에 좌, 우 출력이 RELEASE이고 OFF 상태와 ignition_off 이유가 제공된다. | PC/SIL/Web | SWR-020 |
| OEM-NFR-001 | QM(비기능) | 동일한 입력순서와 초기상태에는 동일한 제어결과가 생성되어야 한다. | 고정 시계로 같은 벡터 1,000회 재생 시 공개결과 해시가 모두 같다. | PC/SIL | SWR-016 |
| OEM-NFR-002 | QM(비기능) | SW는 최근 제어결정 100건을 메모리 내 저장소에 보존하되 영상, 음성, 개인 식별자를 저장하지 않아야 한다. | 101건 입력 후 최신 100건만 순서대로 조회되고 허용 필드만 존재하며 프로세스 재기동 후 데이터가 남지 않는다. | 단위 및 통합 | SWR-010, SWR-011, SWR-012 |

- 용어집: `WorkProducts/Engineering/SoftwareRequirementsAnalysis/glossary.md`.
- 관련 산출물: `SWE1-002_Use Case 명세서.md`, `TRC-001_양방향 요구사항 추적 매트릭스.md`.
- 적용 템플릿: `.claude/skills/requirements-analysis/references/template.md`(TPL-SWE1-001/002, TPL-TRC-001 구조 동기화본).

---

## 13. 일관성 점검 결과

`glossary.md` 대비 용어 통일성과 SWR 간 모순/중복을 스캔한 결과:

- **[용어 정합]** "해제"라는 한국어 표현이 unlock(명령)과 RELEASE(출력)를 혼동시킬 수 있어 glossary.md에 명시적으로 구분 규칙을 두었다. 본 문서 내 모든 SWR 문장은 "unlock 명령"과 "RELEASE 출력"을 구분해 사용했다 — 문제 없음.
- **[다중 Upstream, 모순 아님]** SWR-006 ↔ OEM-SR-002, OEM-FR-003 / SWR-022 ↔ OEM-SR-001, OEM-SR-002: 하나의 SWR이 두 개의 상위 요구에서 도출되는 것은 모순이 아니라 정당한 다중 추적 관계이다.
- **[이전 잠재적 모순 후보 — 본 개정에서 전량 해소]** SWR-005 ↔ SWR-017, SWR-007 ↔ SWR-005, SWR-007 ↔ SWR-018, SWR-018 ↔ SWR-017, SWR-020 ↔ SWR-018, SWR-003 ↔ SWR-001/002: 동시발생 시 출력이 상충할 수 있었던 쌍이었으나, SWR-022(8단계 우선순위)가 신설되어 각 쌍의 우선순위가 결정론적으로 정의되었다. 더 이상 "미결"이 아니며, 각 SWR 본문에 SWR-022 참조와 함께 명시했다.
- **[정정 사항]** SWR-021(Rev 0.1)은 sensor_fault를 "최상위 게이트"로 서술해 crash CONFIRMED까지 무력화하는 결론이었다. 본 개정(Rev 0.2)에서 SWR-022 순위 3으로 재정의되어 crash CONFIRMED(순위1)·화재등(순위2)에는 종속되도록 **정정**되었다. 이는 모순이 아니라 개정에 따른 의도적 변경이며, §8.2 결정 1과 §4.2.4 근거란에 명시했다.
- **[중복 검사]** 23개(013-A/B 세분화 포함, SWR-022 신규 포함) SWR 중 동일 내용을 다른 ID로 중복 기술한 사례는 발견되지 않았다.
- **[용어 불일치 검사]** LOCK/RELEASE/lock/unlock/DEGRADED/FAULT/OFF/override/순위 등 상태·명령·정책 용어가 SWR 전체에서 glossary.md 정의와 일치하게 사용되었음을 확인했다.
- **[신규 식별, 미해결 아님]** §8.3(잔존-3): state 값 중 DEGRADED와 SWR-022의 8단계 순위 간 상호작용이 정의되지 않은 점을 이번 검토에서 새로 식별했다. 표시(SWR-014/015)에만 관련되어 도어 LOCK/RELEASE 결정에는 영향이 없으므로 안전크리티컬 모순으로 분류하지 않으며, 재확인 권장 항목으로 §8.3에 등록했다.

결론: **본 개정 시점 기준 발견된 용어/중복/모순 문제는 없다.** 이전 개정의 8개 우선순위 충돌 쌍은 전량 SWR-022로 해소되었으며, 잔존하는 것은 비안전크리티컬 해석적 사항(§8.3) 2건과 이번에 신규 식별된 표시 관련 사항 1건뿐이다.

---

## 14. A-SPICE / ISO 26262 자체 점검 결과

### A-SPICE (SYS.1/SYS.2/SWE.1 관점)

| 점검 항목 | 결과 | 비고 |
|---|---|---|
| 출처(Source) | Pass | 모든 SWR(001~022)에 Upstream(OEM-XX-xxx) 명시 |
| 우선순위 | Pass | 모든 SWR에 관리용 우선순위(High/Medium) 명시. 런타임 트리거 순위는 SWR-022로 별도 명문화되어 두 개념이 혼동되지 않음 |
| 근거(Rationale) | Pass | 모든 SWR에 근거 서술. 특히 SWR-019/021/022는 해석·정책의 근거와 "원문 근거 없음"이라는 사실을 정직하게 명시 |
| 검증기준(Verification criteria) | Pass | 모든 SWR에 실행 가능한 검증기준(방법+합격기준) 명시. 비기능(SWR-012, SWR-016)은 방법+도구+합격기준 3요소를 모두 충족 |
| 양방향 추적성 | 부분 Pass | Upstream 전량 확보(§11, §12). Downstream(Architecture~Coverage)은 아키텍처/설계/코드/테스트 산출물 부재로 전량 TBD(설계 착수 전 정상 상태) |
| 검토 상태 | Pass | 전체 SWR이 "Draft"로 일관 표기(§3). 이전 개정의 "Draft(부분)" 표기는 §8.2에 기록된 4건의 사용자 결정으로 전량 해소되어 "Draft"로 정정됨 |

### ISO 26262 (안전 관련 SWR: SWR-005, 006, 007, 008, 009, 013-A, 013-B, 021, 022 한정)

| 특성 | 결과 |
|---|---|
| ASIL 등급 명시 | Pass — 전량 ASIL B, OEM 할당을 그대로 승계(재도출 없음). SWR-022는 이 안전요구들 간 상호작용을 규정하므로 ASIL B로 분류 |
| 안전목표와의 연계 | 부분 Pass — Safety Goal/FSR/TSR은 OEM 소유이며 ISO 26262 Part 3/4가 본 프로젝트 범위 밖이므로, OEM-SR-XXX(ASIL B 입력)를 최상위 안전 근거로 그대로 받아들인다. 별도 FSR/TSR 문서를 자체 생성하지 않는다(원문 2절에 따른 의도적 경계) |
| 명확하지 않음(Unambiguous) | Pass — 이전 개정에서 §0(b) 8개 항목으로 인해 "부분 Pass"였던 SWR-005/006/007/008/017/018/020/021이 SWR-022 신설로 전량 명확화되었다 |
| 이해가능/원자적/내적일관성/실현가능/검증가능/추상수준 적절 | Pass(각 SWR 레코드 내 개별 점검 결과 참조). SWR-022는 "정책성 요구"라는 특성상 원자성을 "하나의 정책"으로 판단했음을 §4.2.5에 명시 |
| 추적가능 | Pass(Upstream), 부분 Pass(Downstream TBD) |

**중요 고지**: 본 자체 점검은 실무 보조 목적이며, ISO 26262 각 파트(Part 6 등) 및 Automotive SPICE PAM 원문의 심사·평가를 대체하지 않는다. 특히 ASIL B "할당"은 OEM 입력으로 그대로 수용했으며, HARA/ASIL 결정의 타당성 자체는 본 프로젝트 범위 밖(ISO 26262 Part 3 제외)이다. SWR-022의 우선순위 순서 자체도 OEM 원문에 존재하는 근거가 아니라 사용자 승인에 기반한 프로젝트 정의임을 재차 명시한다(§4.2.5, §8.2).
