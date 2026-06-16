---
id: endpoint.dashboard
kind: Endpoint
title: "GET /dashboard"
definition: 하나의 프로젝트에 대한 전체 현황을 집계하여 반환하는 대시보드 집계 엔드포인트. 대시보드 SPA와 CLI `sdi status`가 소비한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/aggregate.rs"
relatesTo:
  - to: concept.project
    type: reads
    note: "cwd 또는 project 파라미터로 프로젝트를 식별하고 기본 정보를 포함한다."
  - to: concept.plan
    type: reads
    note: "활성 플랜과 전체 플랜 목록을 집계에 포함한다."
  - to: concept.task
    type: reads
    note: "진행 중(in-flight) 태스크와 백로그 태스크 목록, 상태 히스토그램을 집계한다."
  - to: concept.round
    type: reads
    note: "최근 활동 항목 산출 시 라운드 이력을 참조한다."
  - to: concept.usage-budget
    type: reads
    note: "usage rollup(사용량 집계)을 포함한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 프로젝트의 현재 상태를 한 번의 요청으로 파악할 수 있도록 여러 도메인 데이터를 모아 반환한다. 클라이언트가 별도로 여러 엔드포인트를 호출하지 않아도 되도록 집계된 뷰를 제공하는 것이 목적이다. `cwd`(디렉터리 경로)나 `project`(프로젝트 id 또는 key) 중 하나를 지정하면 해당 프로젝트를 식별한 뒤 데이터를 조립하여 반환한다.

## 요청 / 응답

**쿼리 파라미터** (둘 중 하나 필수):

- **cwd**: 로컬 작업 디렉터리 경로. 해당 경로가 등록된 프로젝트를 자동으로 찾는다.
- **project**: 프로젝트 id 또는 key. cwd와 함께 오면 project가 우선한다.

**응답에 포함되는 정보**:

- **프로젝트 기본 정보** — 이름, key, 설명, 활성화 여부.
- **활성 플랜** — 현재 `active` 상태인 플랜 하나. 없으면 null.
- **전체 플랜 목록** — 해당 프로젝트에 속한 모든 플랜(상태 무관).
- **집계 카운트** — 플랜 수, 요구사항 수, 결정 수, 시나리오 수, 라운드 수, 진행 중 태스크 수, 백로그 태스크 수를 각각 숫자로.
- **태스크 상태 히스토그램** — todo / in_progress / done / cancelled / blocked 별 태스크 수.
- **진행 중 태스크 목록** — 현재 in_progress 상태인 태스크들의 요약.
- **백로그 태스크 목록** — todo 상태인 태스크들의 요약.
- **최근 활동** — 가장 최근 20건의 도메인 이벤트 또는 변경 이력 항목.
- **사용량 집계(usage rollup)** — 해당 프로젝트의 LLM 호출 등 사용량 요약.

## 권한 / 제약

인증 불필요. 읽기 전용이며 데이터를 변경하지 않는다. `cwd`나 `project` 중 하나가 반드시 있어야 하며, 매핑되는 프로젝트가 없으면 404를 반환한다.

## provenance

- 대시보드 SPA가 최초 로드 및 SSE 이벤트 수신 후 화면 갱신을 위해 호출한다.
- CLI `sdi status` 명령이 프로젝트 현황을 터미널에 출력할 때 이 엔드포인트의 응답을 사용한다.
- 집계 로직 구현 위치: `sdi-plugin/crates/daemon/src/router/aggregate.rs`.

## 미확정 (OPEN)

- `cwd`와 `project`가 동시에 제공될 때 충돌 처리(project 우선 vs. 에러) 정책이 공식 명세에 없음.
- 최근 활동 20건의 기준이 시간순인지, 특정 이벤트 유형 우선인지 미확인.
- 사용량 집계의 집계 기간(일별/주별/전체) 및 세분화 단위 미확인.
- 응답 캐싱 또는 ETag 지원 여부 미확인.
