---
id: invariant.local-first-no-egress
kind: Invariant
title: 모든 상태와 처리는 로컬에 머문다 — 외부로 나가는 네트워크 요청이 없다
definition: "SDI의 모든 상태(Plan·Scenario·Task·Decision·Round 등)는 로컬 SQLite에만 저장된다. 데몬은 루프백(127.0.0.1)과 유닉스 도메인 소켓에만 바인딩하며, 원격 서버로 데이터를 전송하거나 수신하는 외부 네트워크 요청을 하지 않는다."
governs:
  - platform.sdi
  - domain.project-management
  - domain.knowledge-rag
  - domain.plugin-runtime
implementedIn:
  - "sdi-plugin/crates/daemon/src/main.rs"
  - "sdi-plugin/crates/db/src/paths.rs"
  - "sdi-plugin/docs/ARCHITECTURE.md"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI가 다루는 모든 데이터는 사용자의 기계 안에서만 만들어지고 보관되고 처리된다. `sdid` 데몬은 외부 IP 주소에 바인딩하지 않으며, 어떤 데이터도 외부 서버로 전송하지 않는다. 지식 검색을 위한 임베딩도 `sqlite-vec`을 이용한 온디바이스 벡터 검색으로 처리하며 외부 임베딩 API를 호출하지 않는다.

이 불변식의 이유는 사용자 코드와 의사결정 이력의 프라이버시 보호다. SDI가 관리하는 Plan·Scenario·Decision에는 구현 설계, 아키텍처 결정, 비즈니스 로직이 담겨 있다. 이 데이터가 외부 서비스에 전송되면 사용자가 의식하지 못한 채 민감한 정보가 유출될 수 있다.

공개 바인딩 주소(`0.0.0.0` 등)도 금지된다. 데몬이 공개 주소에 바인딩되면 같은 네트워크의 다른 기기가 SDI의 내부 HTTP API에 접근할 수 있다. HTTP API는 Plan 삭제, Round 활성화 등 파괴적 작업을 포함하므로 인증 없는 공개 노출은 허용되지 않는다.

## 깨지면 무슨 일이 일어나나

데몬이 외부 IP에 바인딩되면 같은 네트워크의 누구든 Plan·Scenario를 읽거나 삭제할 수 있다. 임베딩이 외부 API를 경유하면 코드 컨텍스트와 시나리오 내용이 외부 서버 로그에 남는다. 텔레메트리나 원격 동기화 코드가 몰래 추가되면 사용자가 모르는 사이에 작업 이력이 유출된다. 이 경우 오픈소스 로컬 도구로서의 SDI 정체성이 무너지며, 사용자의 코드베이스와 의사결정 이력의 보안 모델이 붕괴된다.

## 코드에서 어떻게 강제되나

데몬의 리스닝 설정(`crates/daemon/src/main.rs`)은 바인딩 주소를 `127.0.0.1` 또는 유닉스 도메인 소켓으로만 허용한다. 공개 주소 바인딩 시도는 `public-bind-not-allowed`로 거부된다. 외부 HTTP 클라이언트 코드나 원격 동기화 코드는 코드베이스에 존재하지 않는다. `SDI_HOME` 환경변수는 XDG 경로 루트를 재정의할 수 있지만 네트워크 바인딩 주소는 이 환경변수의 영향을 받지 않는다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(로컬 퍼스트 / 외부 이그레스 금지 정책의 결정 엔트리) 연결 필요
