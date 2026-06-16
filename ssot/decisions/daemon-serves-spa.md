---
id: decision.daemon-serves-spa
kind: Decision
title: sdid가 tower-http ServeDir로 대시보드 SPA를 직접 서빙한다
purpose: "대시보드 SPA를 어떻게 사용자에게 노출할지 — 별도 HTTP 서버(Nginx, Vercel 등)로 서빙할지, daemon이 직접 서빙할지"
definition: "daemon(sdid)이 axum + tower-http의 ServeDir을 사용해 plugin/web/dist의 대시보드 SPA를 HTTP로 직접 서빙한다. 별도 정적 서버가 필요 없다."
relatesTo:
  - { to: domain.plugin-runtime, type: governs, note: "플러그인 런타임 안에서 daemon이 웹까지 서빙하는 방식을 이 결정이 정의한다" }
  - { to: platform.sdi, type: impacts, note: "사용자가 대시보드에 접근하는 방법이 이 결정에 의해 정해진다" }
supersedes: []
implementedIn:
  - sdi-plugin/crates/daemon
  - sdi-plugin/plugin/web
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI의 대시보드는 사람 개발자가 시나리오·라운드·결정·자율 정책 상태를 시각적으로 확인하는 표면이다. 이 대시보드를 어떻게 제공할지가 선택지였다. 가능한 방법은 Vercel 같은 외부 호스팅을 쓰거나, 사용자가 별도 HTTP 서버를 띄우거나, daemon이 직접 서빙하는 것이었다.

SDI는 로컬 도구이며 외부 서버 의존 없이 온디바이스에서 완결되어야 한다. daemon(sdid)은 이미 사용자 기기에 상주하면서 SQLite 상태를 관리하고 HTTP API를 제공한다. 이 daemon이 SPA까지 함께 서빙하면 사용자는 추가 서버 설정 없이 대시보드에 접근할 수 있다.

## 결정 (Decision)

daemon(`sdid`)이 axum의 HTTP 서버에 tower-http ServeDir을 마운트하여 `plugin/web/dist/`의 빌드된 SPA를 정적 파일로 서빙한다. daemon이 떠 있으면 대시보드도 자동으로 접근 가능하다. SPA는 Vite + React 19 + Tailwind 4로 빌드된다.

별도 Nginx, Caddy, Vercel 같은 정적 서버는 필요하지 않다. daemon 하나가 API 엔드포인트와 정적 파일 서빙을 모두 담당하는 통합 구조다.

## 근거와 결과 (Consequences)

daemon이 SPA를 서빙하면 세 가지 이점이 생긴다.

첫째, 사용자 경험이 단순해진다. `sdid`를 시작하는 것 하나로 API 서버·웹 대시보드·MCP 서버(unix socket)가 모두 준비된다. 추가 설치·설정이 없다.

둘째, 상태 일관성이 보장된다. API와 SPA가 같은 프로세스에서 나오므로 포트 설정 불일치나 CORS 설정 오류가 없다. SPA가 바라보는 API 주소가 항상 daemon의 주소와 동일하다.

셋째, 플러그인 배포 모델과 맞는다. `plugin/web/dist`는 플러그인 디렉토리 안에 있으므로, 플러그인 설치 시 SPA 번들도 함께 설치된다. 별도 배포 경로가 없다.

대가는 daemon의 책임이 SQLite 관리 + HTTP API + 정적 파일 서빙으로 넓어진다는 것이다. 이는 단일 배포 단위라는 장점과 트레이드오프한 의도적 선택이다.

<!-- provenance: sdi-plugin/CLAUDE.md "sdid 가 tower-http ServeDir 로 SPA 를 직접 서빙한다". sdi-plugin/docs/ARCHITECTURE.md Layout 섹션 "sdi-plugin/plugin/web … Vite/React 19/Tailwind 4". scenario-driven/CLAUDE.md "대시보드 SPA(plugin/web/, Vite/React 19/Tailwind 4). sdid 가 tower-http ServeDir 로 SPA 를 직접 서빙한다." -->
