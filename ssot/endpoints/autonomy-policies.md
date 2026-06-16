---
id: endpoint.autonomy-policies
kind: Endpoint
title: "POST /autonomy_policies, GET /autonomy_policies"
definition: >
  자율성 정책을 생성하거나 갱신하고, 프로젝트에 등록된 정책 목록을 조회하는 엔드포인트.
  POST는 에이전트가 특정 범위에서 어느 수준의 자율성으로 동작할지를 결정하는 정책을 등록하거나 덮어쓴다.
  GET은 지정한 프로젝트의 모든 활성 정책을 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/autonomy_policy.rs"
relatesTo:
  - to: concept.autonomy-policy
    type: mutates
  - to: concept.project
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

에이전트가 주어진 프로젝트나 플랜 범위 안에서 어느 수준(L1~L5)의 자율성으로 행동할 수 있는지를 정책으로 등록하거나 조회하는 엔드포인트다. POST 한 번으로 새 정책을 만들거나 기존 정책을 덮어쓴다(upsert). GET은 프로젝트에 걸린 모든 정책 행을 반환한다. 정책이 변경될 때마다 `autonomy.changed` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /autonomy_policies**

요청 본문에 포함할 수 있는 필드:

- `project_id` (필수): 정책이 적용될 프로젝트 식별자.
- `plan_id` (선택): 정책 범위를 특정 플랜으로 좁힐 때 사용.
- `scope_kind` (필수): 정책이 적용되는 범위의 종류. 예를 들어 플랜 전체, 특정 패턴, 특정 결정 종류 등으로 구분된다.
- `decision_kind` (선택): 정책을 특정 결정 종류에만 적용하고 싶을 때 지정.
- `pattern_kind` (선택): D25에 따라 특정 협업 패턴 종류에 정책을 제한할 때 사용.
- `mode` (필수): 자율성 수준. L1(완전 수동)부터 L5(완전 자율)까지의 열거값.
- `l5_threshold` (선택, 기본값 5): L5 모드에서 자율 전환 조건 점수 임계값. 0~10 범위.
- `pattern_depth_cap` (선택, 기본값 3): 패턴 중첩 깊이 상한. 1~10 범위.
- `plan_single_session_lock` (선택): 플랜 단위 단일 세션 잠금 여부.
- `external_surface` (선택): 외부 시스템 연동 여부 표시.
- `timeout_ms` (선택): 에이전트 동작 타임아웃(밀리초).
- `forced` (선택): D17 강제 L4 불변 조건에 따라 강제 적용 여부를 지정.
- `set_by` (선택, 기본값 `"agent"`): 정책을 설정한 주체. 사람 또는 에이전트.
- `reason` (선택): 정책 설정 이유를 자유 텍스트로 기록.

D17 강제-L4 불변 조건은 저장소 레이어에서 자동으로 검사된다. 조건 위반 시 요청이 거부된다.

응답: 생성 또는 갱신된 정책 행.

**GET /autonomy_policies**

쿼리 파라미터:

- `project_id` (필수): 조회할 프로젝트 식별자.

응답: 해당 프로젝트에 등록된 모든 정책 목록.

## 권한 / 제약

- D17 강제-L4 불변 조건이 저장소 레이어에서 적용된다. `forced` 필드나 특정 `scope_kind` 조합이 이 조건을 위반하면 요청이 실패한다.
- D25에 따라 `pattern_kind` 범위의 정책도 지원된다.
- 정책 변경 시 `autonomy.changed` SSE 이벤트가 발행되므로, 이를 구독하는 모든 컴포넌트에 즉시 반영된다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/autonomy_policy.rs`에서 유추됨. D17(강제-L4 불변), D25(pattern_kind 범위)는 설계 결정 참조.

## 미확정 (OPEN)

- `scope_kind`의 허용 열거값 목록이 코드에서 확인되지 않았다.
- `forced` 필드가 D17 조건을 구체적으로 어떻게 트리거하는지 구현 세부 내용 미확인.
- upsert 충돌 키(composite key 구성) 정확한 컬럼 조합 미확인.
- SSE 이벤트 페이로드 구조 미확인.
