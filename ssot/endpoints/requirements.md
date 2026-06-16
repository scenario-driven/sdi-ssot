---
id: endpoint.requirements
kind: Endpoint
title: "POST /requirements, GET /requirements"
definition: 요구사항을 생성하거나, 특정 플랜에 속한 요구사항 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/requirement.rs"
relatesTo:
  - to: concept.requirement
    type: mutates
  - to: concept.plan
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

요구사항 컬렉션 엔드포인트다. POST로 새 요구사항을 만들고, GET으로 플랜에 속한 요구사항 전체를 나열한다. 요구사항 본문은 스냅샷 전용(SNAPSHOT-ONLY) 형식을 따라야 하며, 변경 이력을 본문 안에 섞는 것을 금지한다(D12 규칙). D23 규칙에 따라 활성 패턴이 없을 경우 서버가 직접 패턴 출처를 자동 할당한다.

## 요청 / 응답

**POST /requirements**
- 요청 본문(JSON):
  - `plan_id`(필수): 이 요구사항을 귀속시킬 플랜 ID.
  - `short_code`(필수): 요구사항을 짧게 식별하는 코드(예: REQ-01).
  - `title`(필수): 요구사항 제목.
  - `body`(필수): 요구사항 본문. 스냅샷 전용 형식이어야 하며, 서버가 삽입 전에 형식을 검증한다.
  - `source`(선택, 기본값 `"snapshot"`): 요구사항 출처 표시.
- D12 검증: 본문에 변경 이력 흔적(이전 내용 기록, 취소선 등)이 포함되어 있으면 거부한다.
- D23 자동 처리: 활성 패턴이 없으면 직접(direct) 패턴 출처를 자동 할당한다.
- 응답: 생성된 요구사항 객체.

**GET /requirements**
- 쿼리 파라미터 `plan_id`(필수): 조회할 플랜 ID. 없으면 요청을 거부한다.
- 응답: 해당 플랜에 속한 요구사항 목록(배열).

## 권한 / 제약

- GET 시 `plan_id` 쿼리 파라미터는 필수다. 플랜 지정 없는 전체 조회는 허용하지 않는다.
- POST 시 본문은 스냅샷 전용 형식 검증을 통과해야 한다. "이전에는 A였으나 현재는 B다" 류의 기술은 거부된다.
- `source` 미지정 시 서버가 `"snapshot"`을 기본값으로 사용한다.

## provenance

sdi-plugin 요구사항 라우터(`requirement.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. D12 스냅샷 검증과 D23 직접 패턴 자동 할당 동작은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- D12 스냅샷 형식 검증의 구체적인 판단 기준(어떤 패턴을 감지하면 거부하는지) 미확인.
- D23 직접 패턴 출처 자동 할당 시 기록되는 필드명과 값 미확인.
- `short_code` 중복 처리 정책(같은 플랜 내 중복 허용 여부) 미확인.
- GET 결과의 정렬 기준(생성 순서, short_code 순서 등) 미확인.
