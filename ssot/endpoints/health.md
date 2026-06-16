---
id: endpoint.health
kind: Endpoint
title: "GET /health"
definition: 데몬(sdid)이 살아 있는지 확인하는 헬스체크 엔드포인트. 응답으로 서비스 이름과 현재 버전을 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/mod.rs"
relatesTo: []
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

sdid 프로세스가 HTTP 요청을 정상적으로 처리할 수 있는 상태인지 확인한다. CLI와 대시보드 SPA가 데몬 기동 여부를 판별할 때 이 엔드포인트를 먼저 호출한다. 인증이나 파라미터 없이 언제든 호출할 수 있으며, 데몬이 기동 중이라면 항상 성공 응답을 돌려준다.

## 요청 / 응답

요청 파라미터 없음. 쿼리스트링·바디·헤더 모두 불필요하다.

성공 시 HTTP 200과 함께 세 가지 정보를 JSON으로 돌려준다.

- **ok**: 항상 `true`. 데몬이 살아 있음을 나타내는 불리언 플래그.
- **service**: 서비스 식별자. 고정값 `"sdid"`.
- **version**: 현재 sdid 바이너리의 시맨틱 버전 문자열 (예: `"0.1.0"`).

데몬이 죽어 있거나 포트가 열리지 않은 경우에는 네트워크 오류(연결 거부)로 응답 자체가 오지 않는다.

## 권한 / 제약

인증 불필요. 로컬호스트에서 데몬 포트로 접근 가능하면 누구나 호출할 수 있다. 사이드이펙트 없음 — 데이터 변경이나 상태 전이를 유발하지 않는다.

## provenance

- CLI `sdi status` 및 `sdi ping` 류 명령의 전제 조건 호출.
- 대시보드 SPA가 초기 로드 시 데몬 연결 확인에 사용.
- 라우터 등록 위치: `sdi-plugin/crates/daemon/src/router/mod.rs`.

## 미확정 (OPEN)

- 데몬이 DB 연결 불량이나 내부 오류 상태일 때도 `ok: true`를 반환하는지, 아니면 별도 에러 응답을 내려야 하는지 정책이 미정.
- 헬스 응답에 데이터베이스 ping 결과나 SSE 브로커 상태 같은 서브시스템 상태를 포함할 계획이 있는지 미확인.
- 버전 문자열이 빌드 타임에 삽입되는지 런타임 설정에서 읽는지 구현 세부 미확인.
