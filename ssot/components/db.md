---
id: component.db
kind: SystemComponent
title: sdi-db (Rust SQLite 저장소 어댑터 크레이트)
purpose: sdi-core의 도메인 엔티티를 SQLite에 읽고 쓰는 단일 저장소 어댑터다. FTS5 키워드 검색을 내장하고, 벡터 검색은 미래 확장으로 유보한다. 모든 영속성 로직을 이 크레이트 한 곳에 격리해 daemon이 저장소 구현 세부사항에 결합되지 않도록 한다.
realizedBy:
  - domain.knowledge-rag
  - domain.project-management
implementedIn:
  - sdi-plugin/crates/db
dependsOn:
  - component.core
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.daemon
relatesTo:
  - to: component.core
    type: depends-on
    note: sdi-core의 도메인 타입을 받아 SQLite row로 매핑한다
  - to: component.daemon
    type: depends-on
    note: daemon이 sdi-db를 유일한 저장소 진입점으로 소비한다
  - to: domain.knowledge-rag
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

sdi-db는 SDI의 영속성 계층 전체를 담당하는 라이브러리 크레이트다. SQLite 데이터베이스 파일을 열고, 스키마 마이그레이션을 적용하고, 모든 도메인 엔티티(Plan, Requirement, Decision, Scenario, Round, Task, AutonomyPolicy, CollaborationPattern 등)를 테이블에 읽고 쓴다.

검색 기능도 이 크레이트가 맡는다. SQLite 내장 전문 검색(FTS5)을 사용해 제목·본문 기준 키워드 검색을 제공한다. 벡터 임베딩 기반 의미 검색은 현재 명시적으로 미구현(deferred) 상태이며, 크레이트 설명서에 그 사실이 기록되어 있다.

연결 풀링은 r2d2를 통해 관리한다. SQLite는 단일 파일 DB이므로 다중 스레드 환경에서 안전한 연결 풀이 필요하다.

데이터 파일 위치는 XDG 경로 불변식(D-XDG)을 따른다. 데이터베이스 파일은 `~/.local/share/sdi/` 아래에 위치하며, 플러그인 설치 경로(`~/.claude/plugins/`) 아래에 놓이는 것을 코드 레벨에서 거부한다.

## 경계와 의존

sdi-db는 sdi-core에 의존해 도메인 타입을 공유받는다. 외부 네트워크 호출이 없으며, 데몬만이 이 크레이트를 소비한다. CLI·MCP는 sdi-db를 직접 사용하지 않고 반드시 데몬의 HTTP API를 통해 저장소에 접근한다. 이 경계가 "데이터는 항상 데몬을 통해 단일 소유"라는 아키텍처 원칙의 기술적 구현이다.

## 통신 패턴

런타임 네트워크 통신 없음. 데몬 프로세스와 같은 메모리 공간에서 함수 호출로 통신한다. SQLite 파일 I/O만 발생한다.

## 하위 서브패키지 (책임 단위)

- **연결 관리**: r2d2 풀 초기화, SQLite 파일 경로 결정(XDG), WAL 모드 설정.
- **마이그레이션**: 버전 추적 테이블과 순서 보장 스키마 적용 로직.
- **엔티티 리포지토리**: 각 도메인 엔티티별 CRUD 쿼리 집합.
- **FTS5 검색**: 전문 검색 가상 테이블 생성과 키워드 쿼리 변환.
- **벡터 검색 (미구현)**: sqlite-vec 또는 동등 수단을 통한 KNN 검색 — 미래 확장 슬롯.

## 미확정 (OPEN)

- [ ] OPEN: 벡터 검색 구현 시점과 sqlite-vec 채택 여부는 결정되지 않았다. 현재 FTS5만 실존한다.
- [ ] OPEN: 마이그레이션 순번 체계(파일명 기준인지, 코드 내 버전 테이블인지)를 db 노드에서 확인 필요.
