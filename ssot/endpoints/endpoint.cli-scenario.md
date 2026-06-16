---
id: endpoint.cli-scenario
kind: Endpoint
title: "CLI `sdi scenario`"
definition: "D5 GWT 시나리오 CRUD — 생성·조회·수정·확정·은퇴·검색·클레임 관리"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/scenario.rs"
  - "sdi-plugin/plugin/commands/scenario.md"
relatesTo:
  - to: "concept.scenario"
    type: "mutates"
  - to: "concept.plan"
    type: "reads"
  - to: "domain.scenario-management"
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

`sdi scenario` 는 SDI 시스템에서 Given/When/Then(GWT) 형식의 행동 시나리오를 관리하는 CLI 명령 그룹이다. 시나리오는 "시스템이 어떻게 동작해야 하는가"를 검증 가능한 명세로 표현하는 1등 시민 객체로, 구현과 회귀 검증의 기준점 역할을 한다.

D5 규칙에 따라 시나리오 생성 시 Given(전제 조건), When(행동 트리거), Then(기대 결과) 세 항목이 모두 필수다. 어느 하나라도 누락되면 생성이 거부된다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: Given/When/Then을 모두 포함한 초안 시나리오를 생성한다. 생성 직후 상태는 초안(draft)이며, 구현 착수 전 확정 절차가 필요하다.

**조회(list/view)**: 플랜 범위 내 시나리오 목록을 상태·태그·연결 태스크 기준으로 필터링하여 반환한다. `view`는 단일 시나리오의 전체 세부 내용을 표시한다.

**수정(update)**: GWT 텍스트, 우선순위, 메타데이터를 수정한다. 확정(confirmed) 상태에서의 수정은 경고를 동반하며, 연결된 라운드 결과에 영향을 미칠 수 있다.

**확정(confirm)**: 초안 상태의 시나리오를 확정(confirmed) 상태로 전환한다. 확정된 시나리오만 라운드 실행 대상이 될 수 있다.

**은퇴(retire / unretire)**: 더 이상 유효하지 않은 시나리오를 은퇴(retired) 상태로 전환한다. 은퇴된 시나리오는 새 라운드에서 실행 대상에서 제외되나, 이력 조회는 유지된다. `unretire`로 복원 가능하다.

**검색(search)**: FTS5 전문검색 엔진을 활용하여 GWT 텍스트 전체를 대상으로 키워드 검색을 수행한다.

**클레임(claim / release)**: D29 에이전트 간 시나리오 소유권 조율 메커니즘이다. 특정 에이전트가 시나리오를 `claim`하면 다른 에이전트가 동시에 같은 시나리오를 수정하거나 실행 착수하는 것을 방지한다. 작업 완료 후 `release`로 소유권을 반납한다.

## 권한 / 제약

- 시나리오는 반드시 활성 플랜에 귀속되어야 한다. 플랜 외부 단독 시나리오는 허용되지 않는다.
- 확정 상태에서 GWT 내용을 수정하면 연결된 라운드 결과의 유효성이 깨질 수 있으므로 시스템이 경고를 발한다.
- 은퇴된 시나리오를 다시 클레임하려면 먼저 `unretire`가 선행되어야 한다.
- `claim` 없이 타 에이전트가 클레임 중인 시나리오를 수정하려 하면 충돌 오류가 반환된다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/scenario.rs` 파일 구조 및 `sdi-plugin/plugin/commands/scenario.md` 스킬 문서를 근거로 추론되었다. D5(GWT 필수), D29(클레임) 규칙 번호는 SDI PRD 기준이며, 구현 코드에서 직접 확인된 것이 아닌 설계 명세 기반 추론이다.

## 미확정 (OPEN)

- `claim` 잠금의 타임아웃 정책(자동 해제 시간)이 명확하게 정의되어 있는지 불확실하다.
- `search`의 FTS5 인덱스 업데이트 시점(실시간 vs. 배치)이 확인되지 않았다.
- `confirm` 이후 GWT 수정 시 연결된 라운드 결과를 자동으로 무효화하는지, 아니면 단순 경고만 발하는지 코드 레벨 검증이 필요하다.
