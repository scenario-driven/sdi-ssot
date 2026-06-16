---
id: endpoint.rounds-results
kind: Endpoint
title: "GET /rounds/:id/results, POST /rounds/:id/results"
definition: 라운드에 속한 시나리오별 검증 판정(verdict)을 조회하거나 기록하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
    type: mutates
  - to: concept.scenario
    type: reads
  - to: concept.regression
    type: mutates
    note: "판정 기록이 회귀 추적의 증거 흔적이 됨"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

라운드 안의 시나리오 검증 결과를 다룬다. GET은 해당 라운드의 모든 판정 목록을 반환하고, POST는 특정 시나리오에 대한 판정을 추가하거나 갱신(upsert)한다.

판정 결과는 회귀 추적의 핵심 증거 기록이 된다. 다음 라운드 활성화 시 D6 기준선 복사에서 이 결과들이 소스로 사용된다.

라운드를 완료하기 위해 모든 시나리오가 `passing` 상태일 필요는 없다. 어떤 결과 조합이 완료 가능 조건인지는 제품 정책이 결정하며, 데몬이 강제하는 게이트가 아니다.

## 요청 / 응답

**GET /rounds/:id/results**

경로 파라미터:

- `id` (필수): 라운드 식별자.

응답: 판정 배열. 각 항목:

- `round_id`: 라운드 식별자.
- `scenario_id`: 시나리오 식별자.
- `result`: 판정 결과 (`passing`, `failing`, `impacted`, `retired` 중 하나).
- `note`: 선택적 설명 메모.
- `evidence_ref`: 선택적 증거 참조.
- `updated_at`: 마지막 갱신 시각.

**POST /rounds/:id/results**

요청 본문(JSON):

- `scenario_id` (필수): 판정을 기록할 시나리오 식별자.
- `result` (필수): 판정 결과 — `passing`, `failing`, `impacted`, `retired` 중 하나.
- `note` (선택): 판정 근거나 맥락을 설명하는 메모.
- `evidence_ref` (선택): 판정을 뒷받침하는 증거 참조(파일 경로, URL 등).

응답: 생성되거나 갱신된 판정 객체.

## 권한 / 제약

- 같은 `round_id` + `scenario_id` 조합으로 POST하면 기존 판정을 덮어쓴다(upsert).
- `result` 값은 `passing`, `failing`, `impacted`, `retired` 네 가지 중 하나여야 한다.
- 라운드 완료가 모든 시나리오 `passing`을 강제하지 않는다. 이 엔드포인트는 데몬 수준의 완료 게이트를 두지 않는다.

## provenance

판정 결과(`result`)의 네 가지 값은 SDI 시나리오 생명주기를 반영한다. `impacted`는 다른 변경에 의해 영향을 받았음을, `retired`는 더 이상 유효하지 않음을 의미한다. D6에 따라 이 판정들이 다음 라운드의 회귀 기준선으로 복사된다.

## 미확정 (OPEN)

- `result` 값으로 `skipped`나 `blocked` 같은 추가 상태가 존재하는지 미확인.
- `evidence_ref`의 형식 규약(URI 스키마, 자유 문자열 여부) 미확인.
- GET 응답에 페이지네이션이 적용되는지 미확인.
