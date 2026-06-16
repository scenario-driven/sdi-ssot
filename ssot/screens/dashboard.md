---
id: screen.dashboard
kind: Screen
title: 대시보드 SPA 진입점 (앱 셸)
purpose: "소로 빌더 또는 코딩 에이전트가 SDI 데몬에 연결해 프로젝트 작업 공간 전체를 탐색하는 진입 화면. 사이드바·탑바·메인 콘텐츠 영역으로 구성되며 프로젝트 선택, 뷰 전환, 디테일 드로어 오버레이를 관장한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/App.tsx
  - sdi-plugin/plugin/web/src/main.tsx
  - sdi-plugin/plugin/web/src/components/shell/AppShell
consumesApi: []
relatesTo:
  - { to: concept.project, type: reads, note: "초기 로드 시 프로젝트 목록을 가져와 첫 번째 프로젝트로 자동 이동한다" }
  - { to: concept.plan, type: reads, note: "활성 플랜 여부를 SSE 이벤트로 지속 갱신한다" }
  - { to: concept.collaboration-pattern, type: reads, note: "탑바 배지에 활성 패턴 수와 direct 안티패턴 존재 여부를 표시한다" }
  - { to: screen.sidebar, type: leads-to, note: "좌측 사이드바 셸 화면" }
  - { to: screen.topbar, type: leads-to, note: "상단 탑바 셸 화면" }
  - { to: screen.summary, type: leads-to }
  - { to: screen.next, type: leads-to }
  - { to: screen.board, type: leads-to }
  - { to: screen.backlog, type: leads-to }
  - { to: screen.timeline, type: leads-to }
  - { to: screen.wiki, type: leads-to }
  - { to: screen.patterns, type: leads-to }
  - { to: screen.detail-drawer, type: leads-to }
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

대시보드 SPA의 최상위 진입점이다. 사용자가 브라우저에서 접속하면 데몬 헬스 체크와 프로젝트 목록을 불러온 뒤 첫 번째 프로젝트의 Summary 뷰로 자동 이동한다. 화면 전체를 AppShell(사이드바 + 탑바 + 메인 콘텐츠)로 감싸며, URL 경로(`/<PROJ-ID>/<view>/<type>/<id>`)를 파싱해 현재 프로젝트·뷰·선택 항목을 결정한다. SSE 이벤트 스트림으로 데몬 이벤트를 실시간 수신하고, 구조적 변경(플랜·라운드·시나리오 생성 등)은 모든 뷰의 데이터를 동시에 갱신한다.

## UI 요소 / 입력 필드

- **AppShell**: 좌측 사이드바, 상단 탑바, 메인 콘텐츠 세 영역의 레이아웃 컨테이너.
- **프로젝트 없음 상태**: 아무 프로젝트도 없으면 "No project selected" 안내와 "Create project" 버튼을 중앙에 표시한다.
- **디테일 드로어 오버레이**: 플랜·시나리오·요구사항·결정·라운드·태스크 중 하나를 선택하면 화면 우측에서 슬라이드로 열리는 오버레이 패널이 나타난다. 좌측 가장자리 드래그로 너비를 조절할 수 있으며, 배경을 클릭하거나 닫기 버튼으로 닫는다.
- **프로젝트 생성 모달**, **프로젝트 설정 모달**, **플랜 생성 모달**: 각각 트리거에 따라 전체 화면 위에 모달로 열린다.
- **커맨드 팔레트**: Cmd+K / Ctrl+K 단축키로 열리는 전역 검색·이동 팔레트.
- **Toast 알림**: 성공·에러 피드백을 화면 하단에 표시한다.

## 표시 데이터 / 호출 API

앱 마운트 시 데몬 `/projects` 엔드포인트로 프로젝트 목록을 가져온다. 이후 `/health`를 10초 간격으로 폴링해 데몬 연결 상태를 표시하고, `/events` SSE로 실시간 이벤트를 구독한다. SSE 이벤트는 태스크 패치(부분 업데이트)와 구조적 변경으로 구분되며, 구조적 변경이 오면 뷰의 refreshKey가 증가해 각 뷰가 자율적으로 최신 데이터를 재조회한다. 활성 CollaborationPattern 개수는 `/patterns/active`를 구조적 변경 및 프로젝트 전환마다 조회해 탑바 배지에 반영한다.

## 상태 / 엣지케이스

- **데몬 오프라인**: 탑바에 "daemon down" 표시, 클릭 시 페이지 리로드로 재연결 시도.
- **프로젝트 없음**: 콘텐츠 영역 중앙에 프로젝트 생성 유도 버튼을 표시한다.
- **URL 직접 접속**: `/<PROJ-ID>/<view>` 형태의 URL을 파싱해 해당 프로젝트와 뷰로 직접 진입한다. PROJ-ID가 URL에 있으면 해당 프로젝트가 즉시 선택된다.
- **드로어 너비**: 조절 값은 로컬 저장소에 보존되며, 최소 360px~최대 90vw 범위로 보정된다.
- **direct 안티패턴 배지**: active 패턴 중 kind가 `direct`인 행이 있으면 탑바 패턴 배지에 빨간 점을 표시한다.

## 미확정 (OPEN)
- [ ] OPEN: SSE 이벤트 스트림 재연결 정책(exponential backoff 등)이 구현됐는지 확인 필요.
