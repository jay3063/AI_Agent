# [PLACEHOLDER] 요구사항 명세 템플릿

> **이 파일은 임시(placeholder) 양식입니다.** 사용자가 실제 사내/프로젝트 템플릿을 제공하면 이 파일 내용을 그대로 그 템플릿으로 교체하십시오. 교체 전까지 이 스킬을 사용하는 모든 산출물 서두에는 "⚠ 확정 템플릿 미반영 — 임시 양식 사용 중"을 명시해야 합니다.

## 요구사항 레코드 (임시 양식)

| 필드 | 설명 |
|---|---|
| ID | 고유 식별자 (`references/traceability-and-consistency.md`의 체계 사용) |
| 레벨 | Stakeholder / System / Software |
| 유형 | Functional / Non-functional |
| 제목 | 한 줄 요약 |
| 본문 | `<주체> shall <조건> <행위> <기준>` 형태의 문장 |
| 근거(Rationale) | 왜 필요한가 |
| 출처(Source) | 이해관계자 요구사항, 안전목표, 규제 등 |
| 우선순위 | High / Medium / Low |
| 안전 관련 여부 / ASIL | 해당 시 ISO 26262 ASIL 등급 |
| 품질특성 (비기능만) | ISO 25010 특성 태그 |
| 검증방안 | 실행 가능한 구체적 방법 + 합격 기준 |
| 다이어그램 (기능만) | 관련 UML/SysML 다이어그램 (Mermaid) |
| Upstream | 상위 요구사항/출처 ID |
| Downstream | 하위 산출물/테스트 케이스 ID (없으면 TBD) |
| 검토 상태 | Draft / Reviewed / Approved |

## 문서 섹션 구성 (임시)

1. 개정 이력
2. 범위 및 용어집 참조
3. Stakeholder Requirements
4. System Requirements (기능/비기능 구분)
5. Software Requirements (기능/비기능 구분)
6. 추적성 매트릭스 (요약, 상세는 `traceability-matrix.md`)
7. 일관성/명확성 점검 결과
8. A-SPICE / ISO 26262 자체 점검 결과
