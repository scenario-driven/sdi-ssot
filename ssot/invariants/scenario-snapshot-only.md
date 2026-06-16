---
id: invariant.scenario-snapshot-only
kind: Invariant
title: Requirement는 스냅샷으로만 존재하며 본문에 변경 이력을 남기지 않는다
definition: "Requirement는 단일 스냅샷(최신 상태만)으로 저장되며, (plan_id, short_code) 키로 ON CONFLICT DO UPDATE 방식으로 덮어쓴다. 본문에 이전 내용 흔적(취소선, '이전엔 A', '기존 안→현재 안' 등)을 남기는 것은 금지되며, 변경 이력은 append-only Decision artifact에서만 표현된다."
governs:
  - concept.requirement
  - domain.requirements
  - domain.scenario-management
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/requirement.rs"
  - "sdi-plugin/crates/core/src/scenario.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

Requirement(요구사항)는 현재 유효한 스냅샷 하나만 존재한다. 같은 `(plan_id, short_code)` 조합의 요구사항을 새로 저장하면 이전 내용이 덮어쓰여진다. 버전 테이블이 없고, 이전 내용을 참조하는 컬럼도 없다.

동시에 Requirement 본문 자체에 변경 흔적(취소선, "이전엔 A였으나 이제는 B", "기존 안: X → 현재 안: Y", "~~구 내용~~" 등)을 남기는 것은 API 수준에서 거부된다.

이 규칙의 이유는 명확하다. 요구사항은 "현재 무엇을 만들어야 하는가"를 표현하는 문서다. 과거 흔적이 본문에 섞이면 LLM이 시나리오를 도출할 때 이전 요구사항과 현재 요구사항을 혼동하고, 폐기된 내용이 여전히 유효한 요구로 해석될 위험이 있다. 변경의 이유와 경위가 필요하면 append-only Decision(`concept.decision`)에 별도로 기록한다.

## 깨지면 무슨 일이 일어나나

Requirement 본문에 과거 흔적이 있으면 LLM이 시나리오를 작성할 때 폐기된 요구사항을 여전히 유효한 것으로 해석한다. 특히 GWT 변환 과정에서 "이전엔 A였다"는 내용이 Given 조건으로 오인될 수 있다. R2+ 회귀에서 regression-runner가 구 내용 기반의 시나리오를 신규 구현에 적용하면 false failing이 발생한다. 버전 분기가 생기면 Plan approve 게이트(`invariant.one-active-plan`)도 어떤 상태의 요구사항을 기준으로 승인했는지 모호해진다.

## 코드에서 어떻게 강제되나

데몬의 요구사항 라우터(`crates/daemon/src/router/requirement.rs`)는 `(plan_id, short_code)` 복합 키에 `ON CONFLICT … DO UPDATE` 전략으로 단일 행을 유지한다. 별도 버전 테이블이 없으며 구 내용은 DB에서 삭제된다. 같은 라우터에서 POST 요청 본문을 파싱할 때 이력 흔적 패턴(`기존 안:`, `~~`, `이전엔`, `원래 안:` 등)이 감지되면 유효성 검사 오류를 반환한다. HOOK_ENFORCEMENT.md #7 인수기준으로도 명시되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D12 SNAPSHOT-ONLY 문서 정책 결정 엔트리) 연결 필요
