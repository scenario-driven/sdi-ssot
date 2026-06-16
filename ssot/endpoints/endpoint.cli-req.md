---
id: endpoint.cli-req
kind: Endpoint
title: "CLI `sdi req`"
definition: "D12 스냅샷 시맨틱 요구사항 CRUD — 현재 시점 진실만 기록, 교체로 이력 표현"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/requirement.rs"
  - "sdi-plugin/plugin/commands/req.md"
relatesTo:
  - to: "concept.requirement"
    type: "mutates"
  - to: "concept.plan"
    type: "reads"
  - to: "domain.requirements"
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

`sdi req` 는 SDI 시스템에서 "무엇을 만들어야 하는가"를 기록하는 요구사항(Requirement) 객체를 관리하는 CLI 명령 그룹이다. 시나리오가 "어떻게 동작해야 하는가"를 검증 명세로 표현한다면, 요구사항은 제품이 충족해야 할 기능적·비기능적 조건을 서술 형태로 담는 상위 맥락 문서다.

D12 스냅샷 원칙을 따른다. 요구사항은 항상 현재 시점의 단일 진실만을 담으며, 이전 내용의 흔적을 본문에 남기지 않는다. 요구사항이 바뀌면 해당 필드를 새 값으로 덮어쓰고, 변경 이력이 필요하다면 별도 의사결정 로그(ADR, Clawket decision)에 기록한다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: 플랜에 귀속된 새 요구사항을 생성한다. 제목, 설명, 우선순위, 출처(stakeholder 또는 도메인 영역)를 입력받는다. 생성 즉시 활성 상태로 등록된다.

**조회(list/view)**: 플랜 범위 내 요구사항 목록을 우선순위·상태·키워드 기준으로 필터링하여 반환한다. `view`는 단일 요구사항의 전체 내용과 연결된 시나리오 목록을 함께 표시한다.

**수정(update)**: fetch-then-overwrite 패턴으로 동작한다. 현재 서버 상태를 먼저 읽어 병합 기준으로 삼고, 사용자가 지정한 필드만 새 값으로 교체한다. 이는 부분 업데이트 시 다른 필드가 의도치 않게 초기화되는 것을 방지한다.

**삭제(delete)**: 요구사항을 영구 삭제한다. 연결된 시나리오가 존재할 경우 삭제 전 경고가 표시된다. 이 명령은 되돌릴 수 없으므로 신중하게 사용해야 한다.

## 권한 / 제약

- 요구사항은 반드시 활성 플랜에 귀속되어야 한다.
- D12 스냅샷 원칙에 따라 요구사항 본문에 "이전에는 A였으나 지금은 B다"와 같은 변경 흔적을 남기는 것은 금지된다. 현재 값만 기록한다.
- 삭제 시 연결된 시나리오의 요구사항 참조가 끊어진다. 시나리오 자체가 삭제되지는 않는다.
- 요구사항과 시나리오 간 명시적 연결 관계는 별도 연결 테이블에서 관리된다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/requirement.rs` 및 `sdi-plugin/plugin/commands/req.md` 파일 구조를 근거로 추론되었다. D12 스냅샷 원칙 및 fetch-then-overwrite 패턴은 SDI PRD 설계 명세 기반 추론이며, 코드에서 직접 확인된 것이 아니다.

## 미확정 (OPEN)

- `delete` 시 연결된 시나리오 처리 정책(경고만인지, 연결 자동 해제인지, 또는 삭제 차단인지)이 코드 레벨에서 확인되지 않았다.
- 요구사항과 시나리오 간 연결은 `sdi req`에서 직접 관리되는지, 별도 명령(예: `sdi req link`)이 존재하는지 불확실하다.
- 버전 관리(소프트 삭제) 방식인지 하드 삭제 방식인지 확인이 필요하다.
