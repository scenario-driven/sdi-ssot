---
id: component.web-dashboard
kind: SystemComponent
title: SDI 웹 대시보드 SPA (Vite/React 19/Tailwind 4)
purpose: sdid가 ServeDir로 직접 서빙하는 단일 페이지 앱으로, 에이전트와 사람 모두가 SDI 플랜·시나리오·라운드·결정의 상태를 시각적으로 파악하고 위키 콘텐츠를 탐색하는 인터페이스다. 별도 웹 서버가 필요 없고 sdid 하나로 API와 SPA가 함께 제공된다.
realizedBy:
  - domain.scenario-management
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - domain.knowledge-rag
implementedIn:
  - sdi-plugin/plugin/web
dependsOn:
  - component.daemon
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
relatesTo:
  - to: component.plugin-shell
    type: belongs-to
    note: plugin/web/ 이 플러그인 셸 안에 있다
  - to: component.daemon
    type: depends-on
    note: 모든 데이터를 sdid HTTP API와 SSE 이벤트 버스로 가져온다
  - to: domain.knowledge-rag
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

웹 대시보드(`plugin/web/`)는 Vite + React 19 + Tailwind CSS 4로 만든 단일 페이지 앱(SPA)이다. 빌드 산출물(`plugin/web/dist/`)을 sdid가 `tower-http ServeDir` 미들웨어로 직접 서빙하므로, 사용자는 별도 웹 서버 설치 없이 `http://localhost:<데몬포트>/`로 대시보드에 접근할 수 있다.

대시보드가 제공하는 뷰는 다섯 가지다. **Summary(요약)** 뷰는 현재 플랜의 전체 진행 상황 — 시나리오 수, 라운드 상태, 최근 결정 — 을 한눈에 보여준다. **Board(보드)** 뷰는 태스크를 칸반 형식으로 상태별로 배치하고, dnd-kit을 통한 드래그 앤 드롭으로 상태 변경을 지원한다. **Backlog(백로그)** 뷰는 전체 시나리오와 연결된 태스크를 우선순위별로 나열한다. **Timeline(타임라인)** 뷰는 라운드와 결정의 시간 순서를 시각화한다. **Wiki(위키)** 뷰는 프로젝트의 `wiki_paths`(기본 `docs/`)에 있는 Markdown 파일을 렌더링해 문서를 대시보드 안에서 탐색하게 한다.

SSE 이벤트 버스를 구독해 데몬 상태 변경을 실시간으로 반영한다.

## 경계와 의존

웹 대시보드는 데몬(`component.daemon`)에만 의존한다. sdid HTTP API를 통해 모든 데이터를 읽고 일부 뮤테이션(태스크 상태 변경 등)을 수행한다. Rust·Node.js 의존이 없으며 브라우저에서 실행되는 순수 SPA다.

SPA 소스는 이전 `sdi-web` 저장소에서 `sdi-plugin/plugin/web/`으로 흡수되었다. `SNAPSHOT.json`에 흡수 시점이 기록되어 있으며, 이후 변경은 이 경로에서만 이뤄진다.

## 통신 패턴

브라우저 ↔ 데몬: HTTP REST API(데이터 fetch/mutate) + SSE(실시간 변경 스트리밍). 브라우저가 데몬과 같은 호스트(로컬)에 있으므로 CORS 이슈가 최소화되고, tower-http CORS 미들웨어로 처리한다.

## 하위 서브패키지 (책임 단위)

- **src/views/**: Summary, Board, Backlog, Timeline, Wiki 각 뷰 컴포넌트.
- **src/lib/api.ts**: 데몬 HTTP API 클라이언트.
- **src/lib/sse.ts**: SSE 이벤트 버스 구독 유틸리티.
- **src/lib/daemonUrl.ts**: 데몬 기본 URL 결정 로직.
- **src/types/entities.ts**: SDI 도메인 엔티티의 TypeScript 타입 정의.
- **dist/**: 빌드 산출물 — sdid가 ServeDir로 서빙하는 최종 정적 파일.

## 미확정 (OPEN)

- [ ] OPEN: 대시보드에서 뮤테이션(태스크 상태 변경, 시나리오 수정 등)이 어느 범위까지 가능한지, 읽기 전용 비율이 얼마인지 코드에서 확인 필요.
