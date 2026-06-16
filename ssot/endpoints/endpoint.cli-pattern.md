---
id: endpoint.cli-pattern
kind: Endpoint
title: "CLI `sdi pattern`"
definition: "D22 CollaborationPattern CRUD + 생애주기(transition/abort) + 중첩 트리 조회"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/pattern.rs"
  - "sdi-plugin/plugin/commands/pattern.md"
relatesTo:
  - to: "concept.collaboration-pattern"
    type: "mutates"
  - to: "concept.pattern-lifecycle"
    type: "mutates"
  - to: "domain.collaboration-patterns"
    type: "belongs-to"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`sdi pattern` 은 SDI 시스템에서 에이전트 협업 방식을 구조화하는 CollaborationPattern 객체를 관리하는 CLI 명령 그룹이다. 협업 패턴은 복수의 에이전트가 하나의 작업 목표를 향해 어떤 형태로 역할을 분담하고 결과를 통합할지 명시하는 구조체다.

D22 설계에서 지원하는 패턴 종류는 Solo(단일 에이전트), Pair(두 에이전트 협업), Fan-out(병렬 분기 후 집계), Pipeline(직렬 파이프라인), Swarm(다수 에이전트 자율 협업) 이다. D26 shape gate가 각 패턴 종류별로 유효한 구조 조건을 검증하며, 조건을 충족하지 않는 패턴 생성은 거부된다.

D24 중첩 허용 정책에 따라 패턴은 다른 패턴의 자식 패턴이 될 수 있어, 복잡한 다단계 협업 구조를 계층적으로 표현할 수 있다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: 패턴 종류(shape)와 참여 에이전트 구성을 입력받아 새 협업 패턴을 생성한다. D26 shape gate가 종류별 구조 유효성을 즉시 검증한다. 예를 들어 Fan-out 패턴은 분기 에이전트가 최소 2개 이상이어야 한다.

**조회(list/show)**: 플랜 또는 라운드 범위 내 협업 패턴 목록을 반환한다. `show`는 단일 패턴의 전체 구성과 참여 에이전트 목록, 현재 생애주기 상태를 표시한다.

**생애주기 전이(transition)**: 패턴의 상태를 다음 단계로 전진시킨다. 패턴 상태 전이는 PATCH lifecycle 라우트를 통해 처리되며, 참여 에이전트의 준비 상태와 조건을 확인한 후 전이가 허용된다.

**중단(abort)**: 실행 중인 패턴을 비정상 종료한다. 참여 에이전트에게 중단 신호가 전달되며, 미완료 하위 패턴도 함께 중단된다.

**트리 조회(tree)**: 패턴의 부모-자식 중첩 구조를 계층적 들여쓰기 형태로 시각화한다. 이 뷰는 클라이언트 사이드에서 데이터를 취합하여 렌더링한다.

## 권한 / 제약

- D26 shape gate: 패턴 종류별 구조 유효성 검증은 생성 시점에 강제된다. 유효하지 않은 구조는 생성이 거부된다.
- D24 중첩: 패턴은 다른 패턴의 자식이 될 수 있으나, 순환 참조는 허용되지 않는다.
- `abort`는 되돌릴 수 없다. 중단된 패턴은 재시작이 아닌 새 패턴 생성으로만 대체할 수 있다.
- Solo 패턴을 Swarm 패턴으로 변환하는 것과 같은 shape 변경은 `transition`으로는 불가하며, 새 패턴 생성이 필요하다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/pattern.rs` 및 `sdi-plugin/plugin/commands/pattern.md` 파일 구조와 SDI PRD D22/D24/D26 설계 규칙을 근거로 추론되었다. shape gate의 구체적 유효성 조건과 `tree` 서브커맨드의 클라이언트 사이드 렌더링 방식은 설계 명세 기반 추론이다.

## 미확정 (OPEN)

- D26 shape gate의 각 패턴 종류별 유효성 조건의 구체적 수치(예: Fan-out 최소 분기 수)가 문서화되어 있지 않다.
- `transition`의 허용 상태 전이 그래프(어떤 상태에서 어떤 상태로 전이 가능한지)가 명확하지 않다.
- `tree` 조회의 중첩 깊이 제한과 대규모 패턴 트리에서의 성능 특성이 확인되지 않았다.
- Swarm 패턴의 구체적 에이전트 자율 협업 프로토콜(에이전트 간 통신 방식)이 별도 명세가 필요하다.
